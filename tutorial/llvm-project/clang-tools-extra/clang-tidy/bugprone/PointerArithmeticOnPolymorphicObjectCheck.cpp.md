# PointerArithmeticOnPolymorphicObjectCheck.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `clang-tools-extra/clang-tidy/bugprone/PointerArithmeticOnPolymorphicObjectCheck.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `PointerArithmeticOnPolymorphicObjectCheck` by registering AST matchers and producing diagnostics/fix-its.
- 用途 (CN): 通过注册 AST 匹配器并生成诊断/修复建议来实现 `PointerArithmeticOnPolymorphicObjectCheck`。

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

### Lines 9-12
```cpp
   9 | #include "PointerArithmeticOnPolymorphicObjectCheck.h"
  10 | #include "clang/AST/ASTContext.h"
  11 | #include "clang/ASTMatchers/ASTMatchFinder.h"
  12 | 
```
- EN: The section imports dependencies such as `PointerArithmeticOnPolymorphicObjectCheck.h`, `clang/AST/ASTContext.h`, `clang/ASTMatchers/ASTMatchFinder.h` needed by this file.
- CN: 本段引入了 `PointerArithmeticOnPolymorphicObjectCheck.h`、`clang/AST/ASTContext.h`、`clang/ASTMatchers/ASTMatchFinder.h` 等依赖，供当前文件使用。

### Lines 13-16
```cpp
  13 | using namespace clang::ast_matchers;
  14 | 
  15 | namespace clang::tidy::bugprone {
  16 | 
```
- EN: Using-directives shorten qualified names so matcher/transformer code reads more directly.
- CN: using 指令缩短了限定名，使匹配器/变换器代码更直接。
- EN: Namespace scopes such as `clang::tidy::bugprone` place the symbols in their intended subsystem.
- CN: 诸如 `clang::tidy::bugprone` 这样的命名空间将符号放入预期的子系统中。

### Lines 17-21
```cpp
  17 | namespace {
  18 | AST_MATCHER(CXXRecordDecl, isAbstract) { return Node.isAbstract(); }
  19 | AST_MATCHER(CXXRecordDecl, isPolymorphic) { return Node.isPolymorphic(); }
  20 | } // namespace
  21 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 22-28
```cpp
  22 | PointerArithmeticOnPolymorphicObjectCheck::
  23 |     PointerArithmeticOnPolymorphicObjectCheck(StringRef Name,
  24 |                                               ClangTidyContext *Context)
  25 |     : ClangTidyCheck(Name, Context),
  26 |       IgnoreInheritedVirtualFunctions(
  27 |           Options.get("IgnoreInheritedVirtualFunctions", false)) {}
  28 | 
```
- EN: This block continues the implementation with declarations or statements centered on `PointerArithmeticOnPolymorphicObjectCheck::`.
- CN: 这一段继续实现，围绕 `PointerArithmeticOnPolymorphicObjectCheck::` 展开声明或语句。

### Lines 29-34
```cpp
  29 | void PointerArithmeticOnPolymorphicObjectCheck::storeOptions(
  30 |     ClangTidyOptions::OptionMap &Opts) {
  31 |   Options.store(Opts, "IgnoreInheritedVirtualFunctions",
  32 |                 IgnoreInheritedVirtualFunctions);
  33 | }
  34 | 
```
- EN: Method definitions such as `PointerArithmeticOnPolymorphicObjectCheck::storeOptions` provide the concrete behavior declared elsewhere.
- CN: 诸如 `PointerArithmeticOnPolymorphicObjectCheck::storeOptions` 的方法定义给出了前面声明的具体行为。

### Lines 35-42
```cpp
  35 | void PointerArithmeticOnPolymorphicObjectCheck::registerMatchers(
  36 |     MatchFinder *Finder) {
  37 |   const auto PolymorphicPointerExpr =
  38 |       expr(hasType(hasCanonicalType(pointerType(pointee(hasCanonicalType(
  39 |                hasDeclaration(cxxRecordDecl(unless(isFinal()), isPolymorphic())
  40 |                                   .bind("pointee"))))))))
  41 |           .bind("pointer");
  42 | 
```
- EN: This block defines AST matcher registration, i.e. the structural patterns the check will look for.
- CN: 这一段定义 AST 匹配器注册，也就是该检查要查找的结构模式。
- EN: Method definitions such as `PointerArithmeticOnPolymorphicObjectCheck::registerMatchers` provide the concrete behavior declared elsewhere.
- CN: 诸如 `PointerArithmeticOnPolymorphicObjectCheck::registerMatchers` 的方法定义给出了前面声明的具体行为。

### Lines 43-51
```cpp
  43 |   const auto PointerExprWithVirtualMethod =
  44 |       expr(hasType(hasCanonicalType(
  45 |                pointerType(pointee(hasCanonicalType(hasDeclaration(
  46 |                    cxxRecordDecl(
  47 |                        unless(isFinal()),
  48 |                        anyOf(hasMethod(isVirtualAsWritten()), isAbstract()))
  49 |                        .bind("pointee"))))))))
  50 |           .bind("pointer");
  51 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const auto PointerExprWithVirtualMethod =`.
- CN: 这一段继续实现，围绕 `const auto PointerExprWithVirtualMethod =` 展开声明或语句。

### Lines 52-55
```cpp
  52 |   const auto SelectedPointerExpr = IgnoreInheritedVirtualFunctions
  53 |                                        ? PointerExprWithVirtualMethod
  54 |                                        : PolymorphicPointerExpr;
  55 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const auto SelectedPointerExpr = IgnoreInheritedVirtualFunct`.
- CN: 这一段继续实现，围绕 `const auto SelectedPointerExpr = IgnoreInheritedVirtualFunct` 展开声明或语句。

### Lines 56-59
```cpp
  56 |   const auto ArraySubscript =
  57 |       expr(arraySubscriptExpr(hasBase(SelectedPointerExpr)),
  58 |            unless(isInstantiationDependent()));
  59 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const auto ArraySubscript =`.
- CN: 这一段继续实现，围绕 `const auto ArraySubscript =` 展开声明或语句。

### Lines 60-63
```cpp
  60 |   const auto BinaryOperators =
  61 |       binaryOperator(hasAnyOperatorName("+", "-", "+=", "-="),
  62 |                      hasEitherOperand(SelectedPointerExpr));
  63 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const auto BinaryOperators =`.
- CN: 这一段继续实现，围绕 `const auto BinaryOperators =` 展开声明或语句。

### Lines 64-71
```cpp
  64 |   const auto UnaryOperators = unaryOperator(
  65 |       hasAnyOperatorName("++", "--"), hasUnaryOperand(SelectedPointerExpr));
  66 | 
  67 |   Finder->addMatcher(ArraySubscript, this);
  68 |   Finder->addMatcher(BinaryOperators, this);
  69 |   Finder->addMatcher(UnaryOperators, this);
  70 | }
  71 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const auto UnaryOperators = unaryOperator(`.
- CN: 这一段继续实现，围绕 `const auto UnaryOperators = unaryOperator(` 展开声明或语句。

### Lines 72-76
```cpp
  72 | void PointerArithmeticOnPolymorphicObjectCheck::check(
  73 |     const MatchFinder::MatchResult &Result) {
  74 |   const auto *PointerExpr = Result.Nodes.getNodeAs<Expr>("pointer");
  75 |   const auto *PointeeDecl = Result.Nodes.getNodeAs<CXXRecordDecl>("pointee");
  76 | 
```
- EN: Method definitions such as `PointerArithmeticOnPolymorphicObjectCheck::check` provide the concrete behavior declared elsewhere.
- CN: 诸如 `PointerArithmeticOnPolymorphicObjectCheck::check` 的方法定义给出了前面声明的具体行为。

### Lines 77-83
```cpp
  77 |   diag(PointerExpr->getBeginLoc(),
  78 |        "pointer arithmetic on polymorphic object of type %0 can result in "
  79 |        "undefined behavior if the dynamic type differs from the pointer type")
  80 |       << PointeeDecl << PointerExpr->getSourceRange();
  81 | }
  82 | 
  83 | } // namespace clang::tidy::bugprone
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

## Key Concepts / 关键概念
- clang-tidy check lifecycle / clang-tidy 检查生命周期
- AST matcher DSL / AST 匹配器 DSL
- diagnostics and fix-its / 诊断与修复建议
- namespace scoping / 命名空间作用域

## Dependencies / 依赖关系
- EN: Direct includes: `PointerArithmeticOnPolymorphicObjectCheck.h`, `clang/AST/ASTContext.h`, `clang/ASTMatchers/ASTMatchFinder.h`.
- CN: 直接包含依赖: `PointerArithmeticOnPolymorphicObjectCheck.h`、`clang/AST/ASTContext.h`、`clang/ASTMatchers/ASTMatchFinder.h`。
- EN: Namespace context: `clang::tidy::bugprone`.
- CN: 命名空间上下文: `clang::tidy::bugprone`。
