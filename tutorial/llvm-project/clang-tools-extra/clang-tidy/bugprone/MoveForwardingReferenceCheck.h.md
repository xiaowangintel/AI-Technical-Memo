# MoveForwardingReferenceCheck.h — Code Analysis / 代码分析
## Source / 来源
- File: `clang-tools-extra/clang-tidy/bugprone/MoveForwardingReferenceCheck.h`
- Repository: `llvm-project`
- Purpose (EN): Declares the clang-tidy check class `MoveForwardingReferenceCheck` and the callbacks it overrides.
- 用途 (CN): 声明 clang-tidy 检查类 `MoveForwardingReferenceCheck` 以及它重写的回调。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8
```cpp
   1 | //===----------------------------------------------------------------------===//
   2 | //
   3 | // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4 | // See https://llvm.org/LICENSE.txt for license information.
   5 | // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6 | //
   7 | //===----------------------------------------------------------------------===//
   8 | 
```
- EN: Standard LLVM banner and licensing notice for the file.
- CN: 这是文件的标准 LLVM 版权与许可证说明。

### Lines 9-13
```cpp
   9 | #ifndef LLVM_CLANG_TOOLS_EXTRA_CLANG_TIDY_BUGPRONE_MOVEFORWARDINGREFERENCECHECK_H
  10 | #define LLVM_CLANG_TOOLS_EXTRA_CLANG_TIDY_BUGPRONE_MOVEFORWARDINGREFERENCECHECK_H
  11 | 
  12 | #include "../ClangTidyCheck.h"
  13 | 
```
- EN: This block opens the header guard that prevents multiple inclusion.
- CN: 这一段开启头文件保护，避免被重复包含。
- EN: The section imports dependencies such as `../ClangTidyCheck.h` needed by this file.
- CN: 本段引入了 `../ClangTidyCheck.h` 等依赖，供当前文件使用。

### Lines 14-23
```cpp
  14 | namespace clang::tidy::bugprone {
  15 | 
  16 | /// The check warns if std::move is applied to a forwarding reference (i.e. an
  17 | /// rvalue reference of a function template argument type).
  18 | ///
  19 | /// If a developer is unaware of the special rules for template argument
  20 | /// deduction on forwarding references, it will seem reasonable to apply
  21 | /// std::move to the forwarding reference, in the same way that this would be
  22 | /// done for a "normal" rvalue reference.
  23 | ///
```
- EN: Namespace scopes such as `clang::tidy::bugprone` place the symbols in their intended subsystem.
- CN: 诸如 `clang::tidy::bugprone` 这样的命名空间将符号放入预期的子系统中。

### Lines 24-33
```cpp
  24 | /// This has a consequence that is usually unwanted and possibly surprising: if
  25 | /// the function that takes the forwarding reference as its parameter is called
  26 | /// with an lvalue, that lvalue will be moved from (and hence placed into an
  27 | /// indeterminate state) even though no std::move was applied to the lvalue at
  28 | /// the call site.
  29 | //
  30 | /// The check suggests replacing the std::move with a std::forward.
  31 | ///
  32 | /// For the user-facing documentation see:
  33 | /// https://clang.llvm.org/extra/clang-tidy/checks/bugprone/move-forwarding-reference.html
```
- EN: Comment lines document the intent of the declarations or implementation below.
- CN: 这些注释说明了后续声明或实现的意图。

### Lines 34-43
```cpp
  34 | class MoveForwardingReferenceCheck : public ClangTidyCheck {
  35 | public:
  36 |   MoveForwardingReferenceCheck(StringRef Name, ClangTidyContext *Context)
  37 |       : ClangTidyCheck(Name, Context) {}
  38 |   bool isLanguageVersionSupported(const LangOptions &LangOpts) const override {
  39 |     return LangOpts.CPlusPlus11;
  40 |   }
  41 |   void registerMatchers(ast_matchers::MatchFinder *Finder) override;
  42 |   void check(const ast_matchers::MatchFinder::MatchResult &Result) override;
  43 | };
```
- EN: It declares class `MoveForwardingReferenceCheck` and derives from `ClangTidyCheck`, which defines the framework contract it follows.
- CN: 这里声明类 `MoveForwardingReferenceCheck`，并继承自 `ClangTidyCheck`，说明它遵循的框架契约。
- EN: This block defines AST matcher registration, i.e. the structural patterns the check will look for.
- CN: 这一段定义 AST 匹配器注册，也就是该检查要查找的结构模式。
- EN: This logic gates the check on a specific language mode so it only runs when the feature is available.
- CN: 这里按特定语言模式进行门控，确保检查只在相应特性可用时运行。

### Lines 44-47
```cpp
  44 | 
  45 | } // namespace clang::tidy::bugprone
  46 | 
  47 | #endif // LLVM_CLANG_TOOLS_EXTRA_CLANG_TIDY_BUGPRONE_MOVEFORWARDINGREFERENCECHECK_H
```
- EN: This closes the header guard and ends the header definition.
- CN: 这里关闭头文件保护，结束头文件定义。

## Key Concepts / 关键概念
- clang-tidy check lifecycle / clang-tidy 检查生命周期
- AST matcher DSL / AST 匹配器 DSL
- language-version gating / 语言版本门控
- header include guards / 头文件包含保护
- namespace scoping / 命名空间作用域

## Dependencies / 依赖关系
- EN: Direct includes: `../ClangTidyCheck.h`.
- CN: 直接包含依赖: `../ClangTidyCheck.h`。
- EN: Framework base types: `ClangTidyCheck`.
- CN: 框架基类: `ClangTidyCheck`。
- EN: Namespace context: `clang::tidy::bugprone`.
- CN: 命名空间上下文: `clang::tidy::bugprone`。
