# EasilySwappableParametersCheck.h — Code Analysis / 代码分析
## Source / 来源
- File: `clang-tools-extra/clang-tidy/bugprone/EasilySwappableParametersCheck.h`
- Repository: `llvm-project`
- Purpose (EN): Declares the clang-tidy check class `EasilySwappableParametersCheck` and the callbacks it overrides.
- 用途 (CN): 声明 clang-tidy 检查类 `EasilySwappableParametersCheck` 以及它重写的回调。

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
   9 | #ifndef LLVM_CLANG_TOOLS_EXTRA_CLANG_TIDY_BUGPRONE_EASILYSWAPPABLEPARAMETERSCHECK_H
  10 | #define LLVM_CLANG_TOOLS_EXTRA_CLANG_TIDY_BUGPRONE_EASILYSWAPPABLEPARAMETERSCHECK_H
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
  16 | /// Finds function definitions where parameters of convertible types follow
  17 | /// each other directly, making call sites prone to calling the function with
  18 | /// swapped (or badly ordered) arguments.
  19 | ///
  20 | /// For the user-facing documentation see:
  21 | /// https://clang.llvm.org/extra/clang-tidy/checks/bugprone/easily-swappable-parameters.html
  22 | class EasilySwappableParametersCheck : public ClangTidyCheck {
  23 | public:
```
- EN: Namespace scopes such as `clang::tidy::bugprone` place the symbols in their intended subsystem.
- CN: 诸如 `clang::tidy::bugprone` 这样的命名空间将符号放入预期的子系统中。
- EN: It declares class `EasilySwappableParametersCheck` and derives from `ClangTidyCheck`, which defines the framework contract it follows.
- CN: 这里声明类 `EasilySwappableParametersCheck`，并继承自 `ClangTidyCheck`，说明它遵循的框架契约。

### Lines 24-28
```cpp
  24 |   EasilySwappableParametersCheck(StringRef Name, ClangTidyContext *Context);
  25 |   void registerMatchers(ast_matchers::MatchFinder *Finder) override;
  26 |   void check(const ast_matchers::MatchFinder::MatchResult &Result) override;
  27 |   void storeOptions(ClangTidyOptions::OptionMap &Opts) override;
  28 | 
```
- EN: This block defines AST matcher registration, i.e. the structural patterns the check will look for.
- CN: 这一段定义 AST 匹配器注册，也就是该检查要查找的结构模式。

### Lines 29-32
```cpp
  29 |   /// The minimum length of an adjacent swappable parameter range required for
  30 |   /// a diagnostic.
  31 |   const std::size_t MinimumLength;
  32 | 
```
- EN: This block continues the implementation with declarations or statements centered on `/// The minimum length of an adjacent swappable parameter ra`.
- CN: 这一段继续实现，围绕 `/// The minimum length of an adjacent swappable parameter ra` 展开声明或语句。

### Lines 33-39
```cpp
  33 |   /// The parameter names (as written in the source text) to be ignored.
  34 |   const std::vector<StringRef> IgnoredParameterNames;
  35 | 
  36 |   /// The parameter typename suffixes (as written in the source code) to be
  37 |   /// ignored.
  38 |   const std::vector<StringRef> IgnoredParameterTypeSuffixes;
  39 | 
```
- EN: This block continues the implementation with declarations or statements centered on `/// The parameter names (as written in the source text) to b`.
- CN: 这一段继续实现，围绕 `/// The parameter names (as written in the source text) to b` 展开声明或语句。

### Lines 40-43
```cpp
  40 |   /// Whether to consider differently qualified versions of the same type
  41 |   /// mixable.
  42 |   const bool QualifiersMix;
  43 | 
```
- EN: This block continues the implementation with declarations or statements centered on `/// Whether to consider differently qualified versions of th`.
- CN: 这一段继续实现，围绕 `/// Whether to consider differently qualified versions of th` 展开声明或语句。

### Lines 44-48
```cpp
  44 |   /// Whether to model implicit conversions "in full" (conditions apply)
  45 |   /// during analysis and consider types that are implicitly convertible to
  46 |   /// one another mixable.
  47 |   const bool ModelImplicitConversions;
  48 | 
```
- EN: This block continues the implementation with declarations or statements centered on `/// Whether to model implicit conversions "in full" (conditi`.
- CN: 这一段继续实现，围绕 `/// Whether to model implicit conversions "in full" (conditi` 展开声明或语句。

### Lines 49-52
```cpp
  49 |   /// If enabled, diagnostics for parameters that are used together in a
  50 |   /// similar way are not emitted.
  51 |   const bool SuppressParametersUsedTogether;
  52 | 
```
- EN: This block continues the implementation with declarations or statements centered on `/// If enabled, diagnostics for parameters that are used tog`.
- CN: 这一段继续实现，围绕 `/// If enabled, diagnostics for parameters that are used tog` 展开声明或语句。

### Lines 53-59
```cpp
  53 |   /// The number of characters two parameter names might be dissimilar at
  54 |   /// either end for the report about the parameters to be silenced.
  55 |   /// E.g. the names "LHS" and "RHS" are 1-dissimilar suffixes of each other,
  56 |   /// while "Text1" and "Text2" are 1-dissimilar prefixes of each other.
  57 |   const std::size_t NamePrefixSuffixSilenceDissimilarityThreshold;
  58 | };
  59 | 
```
- EN: This block continues the implementation with declarations or statements centered on `/// The number of characters two parameter names might be di`.
- CN: 这一段继续实现，围绕 `/// The number of characters two parameter names might be di` 展开声明或语句。

### Lines 60-62
```cpp
  60 | } // namespace clang::tidy::bugprone
  61 | 
  62 | #endif // LLVM_CLANG_TOOLS_EXTRA_CLANG_TIDY_BUGPRONE_EASILYSWAPPABLEPARAMETERSCHECK_H
```
- EN: This closes the header guard and ends the header definition.
- CN: 这里关闭头文件保护，结束头文件定义。

## Key Concepts / 关键概念
- clang-tidy check lifecycle / clang-tidy 检查生命周期
- AST matcher DSL / AST 匹配器 DSL
- header include guards / 头文件包含保护
- namespace scoping / 命名空间作用域

## Dependencies / 依赖关系
- EN: Direct includes: `../ClangTidyCheck.h`.
- CN: 直接包含依赖: `../ClangTidyCheck.h`。
- EN: Framework base types: `ClangTidyCheck`.
- CN: 框架基类: `ClangTidyCheck`。
- EN: Namespace context: `clang::tidy::bugprone`.
- CN: 命名空间上下文: `clang::tidy::bugprone`。
