# DurationSubtractionCheck.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `clang-tools-extra/clang-tidy/abseil/DurationSubtractionCheck.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `DurationSubtractionCheck` by registering AST matchers and producing diagnostics/fix-its.
- 用途 (CN): 通过注册 AST 匹配器并生成诊断/修复建议来实现 `DurationSubtractionCheck`。

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

### Lines 9-15
```cpp
   9 | #include "DurationSubtractionCheck.h"
  10 | #include "DurationRewriter.h"
  11 | #include "clang/AST/ASTContext.h"
  12 | #include "clang/ASTMatchers/ASTMatchFinder.h"
  13 | #include "clang/Tooling/FixIt.h"
  14 | #include <optional>
  15 | 
```
- EN: The section imports dependencies such as `DurationSubtractionCheck.h`, `DurationRewriter.h`, `clang/AST/ASTContext.h`, `clang/ASTMatchers/ASTMatchFinder.h` needed by this file.
- CN: 本段引入了 `DurationSubtractionCheck.h`、`DurationRewriter.h`、`clang/AST/ASTContext.h`、`clang/ASTMatchers/ASTMatchFinder.h` 等依赖，供当前文件使用。

### Lines 16-19
```cpp
  16 | using namespace clang::ast_matchers;
  17 | 
  18 | namespace clang::tidy::abseil {
  19 | 
```
- EN: Using-directives shorten qualified names so matcher/transformer code reads more directly.
- CN: using 指令缩短了限定名，使匹配器/变换器代码更直接。
- EN: Namespace scopes such as `clang::tidy::abseil` place the symbols in their intended subsystem.
- CN: 诸如 `clang::tidy::abseil` 这样的命名空间将符号放入预期的子系统中。

### Lines 20-29
```cpp
  20 | void DurationSubtractionCheck::registerMatchers(MatchFinder *Finder) {
  21 |   Finder->addMatcher(
  22 |       binaryOperator(
  23 |           hasOperatorName("-"),
  24 |           hasLHS(callExpr(callee(functionDecl(durationConversionFunction())
  25 |                                      .bind("function_decl")),
  26 |                           hasArgument(0, expr().bind("lhs_arg")))))
  27 |           .bind("binop"),
  28 |       this);
  29 | }
```
- EN: This block defines AST matcher registration, i.e. the structural patterns the check will look for.
- CN: 这一段定义 AST 匹配器注册，也就是该检查要查找的结构模式。
- EN: Method definitions such as `DurationSubtractionCheck::registerMatchers` provide the concrete behavior declared elsewhere.
- CN: 诸如 `DurationSubtractionCheck::registerMatchers` 的方法定义给出了前面声明的具体行为。

### Lines 30-34
```cpp
  30 | 
  31 | void DurationSubtractionCheck::check(const MatchFinder::MatchResult &Result) {
  32 |   const auto *Binop = Result.Nodes.getNodeAs<BinaryOperator>("binop");
  33 |   const auto *FuncDecl = Result.Nodes.getNodeAs<FunctionDecl>("function_decl");
  34 | 
```
- EN: Method definitions such as `DurationSubtractionCheck::check` provide the concrete behavior declared elsewhere.
- CN: 诸如 `DurationSubtractionCheck::check` 的方法定义给出了前面声明的具体行为。

### Lines 35-38
```cpp
  35 |   // Don't try to replace things inside of macro definitions.
  36 |   if (Binop->getExprLoc().isMacroID() || Binop->getExprLoc().isInvalid())
  37 |     return;
  38 | 
```
- EN: This block continues the implementation with declarations or statements centered on `// Don't try to replace things inside of macro definitions.`.
- CN: 这一段继续实现，围绕 `// Don't try to replace things inside of macro definitions.` 展开声明或语句。

### Lines 39-43
```cpp
  39 |   std::optional<DurationScale> Scale =
  40 |       getScaleForDurationInverse(FuncDecl->getName());
  41 |   if (!Scale)
  42 |     return;
  43 | 
```
- EN: This block continues the implementation with declarations or statements centered on `std::optional<DurationScale> Scale =`.
- CN: 这一段继续实现，围绕 `std::optional<DurationScale> Scale =` 展开声明或语句。

### Lines 44-48
```cpp
  44 |   const std::string RhsReplacement =
  45 |       rewriteExprFromNumberToDuration(Result, *Scale, Binop->getRHS());
  46 | 
  47 |   const Expr *LhsArg = Result.Nodes.getNodeAs<Expr>("lhs_arg");
  48 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const std::string RhsReplacement =`.
- CN: 这一段继续实现，围绕 `const std::string RhsReplacement =` 展开声明或语句。

### Lines 49-58
```cpp
  49 |   diag(Binop->getBeginLoc(), "perform subtraction in the duration domain")
  50 |       << FixItHint::CreateReplacement(
  51 |              Binop->getSourceRange(),
  52 |              (llvm::Twine("absl::") + FuncDecl->getName() + "(" +
  53 |               tooling::fixit::getText(*LhsArg, *Result.Context) + " - " +
  54 |               RhsReplacement + ")")
  55 |                  .str());
  56 | }
  57 | 
  58 | } // namespace clang::tidy::abseil
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。
- EN: Method definitions such as `FixItHint::CreateReplacement` provide the concrete behavior declared elsewhere.
- CN: 诸如 `FixItHint::CreateReplacement` 的方法定义给出了前面声明的具体行为。

## Key Concepts / 关键概念
- AST matcher DSL / AST 匹配器 DSL
- diagnostics and fix-its / 诊断与修复建议
- namespace scoping / 命名空间作用域

## Dependencies / 依赖关系
- EN: Direct includes: `DurationSubtractionCheck.h`, `DurationRewriter.h`, `clang/AST/ASTContext.h`, `clang/ASTMatchers/ASTMatchFinder.h`, `clang/Tooling/FixIt.h`, `optional`.
- CN: 直接包含依赖: `DurationSubtractionCheck.h`、`DurationRewriter.h`、`clang/AST/ASTContext.h`、`clang/ASTMatchers/ASTMatchFinder.h`、`clang/Tooling/FixIt.h`、`optional`。
- EN: Namespace context: `clang::tidy::abseil`.
- CN: 命名空间上下文: `clang::tidy::abseil`。
