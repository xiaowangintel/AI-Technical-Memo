# MisplacedPointerArithmeticInAllocCheck.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `clang-tools-extra/clang-tidy/bugprone/MisplacedPointerArithmeticInAllocCheck.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `MisplacedPointerArithmeticInAllocCheck` by registering AST matchers and producing diagnostics/fix-its.
- 用途 (CN): 通过注册 AST 匹配器并生成诊断/修复建议来实现 `MisplacedPointerArithmeticInAllocCheck`。

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
   9 | #include "MisplacedPointerArithmeticInAllocCheck.h"
  10 | #include "clang/AST/ASTContext.h"
  11 | #include "clang/ASTMatchers/ASTMatchFinder.h"
  12 | #include "clang/Lex/Lexer.h"
  13 | 
```
- EN: The section imports dependencies such as `MisplacedPointerArithmeticInAllocCheck.h`, `clang/AST/ASTContext.h`, `clang/ASTMatchers/ASTMatchFinder.h`, `clang/Lex/Lexer.h` needed by this file.
- CN: 本段引入了 `MisplacedPointerArithmeticInAllocCheck.h`、`clang/AST/ASTContext.h`、`clang/ASTMatchers/ASTMatchFinder.h`、`clang/Lex/Lexer.h` 等依赖，供当前文件使用。

### Lines 14-17
```cpp
  14 | using namespace clang::ast_matchers;
  15 | 
  16 | namespace clang::tidy::bugprone {
  17 | 
```
- EN: Using-directives shorten qualified names so matcher/transformer code reads more directly.
- CN: using 指令缩短了限定名，使匹配器/变换器代码更直接。
- EN: Namespace scopes such as `clang::tidy::bugprone` place the symbols in their intended subsystem.
- CN: 诸如 `clang::tidy::bugprone` 这样的命名空间将符号放入预期的子系统中。

### Lines 18-23
```cpp
  18 | void MisplacedPointerArithmeticInAllocCheck::registerMatchers(
  19 |     MatchFinder *Finder) {
  20 |   const auto AllocFunc =
  21 |       functionDecl(hasAnyName("::malloc", "std::malloc", "::alloca", "::calloc",
  22 |                               "std::calloc", "::realloc", "std::realloc"));
  23 | 
```
- EN: This block defines AST matcher registration, i.e. the structural patterns the check will look for.
- CN: 这一段定义 AST 匹配器注册，也就是该检查要查找的结构模式。
- EN: Method definitions such as `MisplacedPointerArithmeticInAllocCheck::registerMatchers` provide the concrete behavior declared elsewhere.
- CN: 诸如 `MisplacedPointerArithmeticInAllocCheck::registerMatchers` 的方法定义给出了前面声明的具体行为。

### Lines 24-28
```cpp
  24 |   const auto AllocFuncPtr =
  25 |       varDecl(hasType(isConstQualified()),
  26 |               hasInitializer(ignoringParenImpCasts(
  27 |                   declRefExpr(hasDeclaration(AllocFunc)))));
  28 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const auto AllocFuncPtr =`.
- CN: 这一段继续实现，围绕 `const auto AllocFuncPtr =` 展开声明或语句。

### Lines 29-32
```cpp
  29 |   const auto AdditiveOperator = binaryOperator(hasAnyOperatorName("+", "-"));
  30 | 
  31 |   const auto IntExpr = expr(hasType(isInteger()));
  32 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const auto AdditiveOperator = binaryOperator(hasAnyOperatorN`.
- CN: 这一段继续实现，围绕 `const auto AdditiveOperator = binaryOperator(hasAnyOperatorN` 展开声明或语句。

### Lines 33-42
```cpp
  33 |   const auto AllocCall = callExpr(callee(decl(anyOf(AllocFunc, AllocFuncPtr))));
  34 | 
  35 |   Finder->addMatcher(
  36 |       binaryOperator(
  37 |           AdditiveOperator,
  38 |           hasLHS(anyOf(AllocCall, castExpr(hasSourceExpression(AllocCall)))),
  39 |           hasRHS(IntExpr))
  40 |           .bind("PtrArith"),
  41 |       this);
  42 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const auto AllocCall = callExpr(callee(decl(anyOf(AllocFunc,`.
- CN: 这一段继续实现，围绕 `const auto AllocCall = callExpr(callee(decl(anyOf(AllocFunc,` 展开声明或语句。

### Lines 43-50
```cpp
  43 |   const auto New = cxxNewExpr(unless(isArray()));
  44 | 
  45 |   Finder->addMatcher(binaryOperator(AdditiveOperator,
  46 |                                     hasLHS(anyOf(New, castExpr(New))),
  47 |                                     hasRHS(IntExpr))
  48 |                          .bind("PtrArith"),
  49 |                      this);
  50 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const auto New = cxxNewExpr(unless(isArray()));`.
- CN: 这一段继续实现，围绕 `const auto New = cxxNewExpr(unless(isArray()));` 展开声明或语句。

### Lines 51-59
```cpp
  51 |   const auto ArrayNew = cxxNewExpr(isArray());
  52 | 
  53 |   Finder->addMatcher(binaryOperator(AdditiveOperator,
  54 |                                     hasLHS(anyOf(ArrayNew, castExpr(ArrayNew))),
  55 |                                     hasRHS(IntExpr))
  56 |                          .bind("PtrArith"),
  57 |                      this);
  58 | }
  59 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const auto ArrayNew = cxxNewExpr(isArray());`.
- CN: 这一段继续实现，围绕 `const auto ArrayNew = cxxNewExpr(isArray());` 展开声明或语句。

### Lines 60-65
```cpp
  60 | void MisplacedPointerArithmeticInAllocCheck::check(
  61 |     const MatchFinder::MatchResult &Result) {
  62 |   const auto *PtrArith = Result.Nodes.getNodeAs<BinaryOperator>("PtrArith");
  63 |   const Expr *AllocExpr = PtrArith->getLHS()->IgnoreParenCasts();
  64 |   std::string CallName;
  65 | 
```
- EN: Method definitions such as `MisplacedPointerArithmeticInAllocCheck::check` provide the concrete behavior declared elsewhere.
- CN: 诸如 `MisplacedPointerArithmeticInAllocCheck::check` 的方法定义给出了前面声明的具体行为。

### Lines 66-75
```cpp
  66 |   if (const auto *Call = dyn_cast<CallExpr>(AllocExpr)) {
  67 |     const NamedDecl *Func = Call->getDirectCallee();
  68 |     if (!Func)
  69 |       Func = cast<NamedDecl>(Call->getCalleeDecl());
  70 |     CallName = Func->getName().str();
  71 |   } else {
  72 |     const auto *New = cast<CXXNewExpr>(AllocExpr);
  73 |     if (New->isArray()) {
  74 |       CallName = "operator new[]";
  75 |     } else {
```
- EN: This block continues the implementation with declarations or statements centered on `if (const auto *Call = dyn_cast<CallExpr>(AllocExpr)) {`.
- CN: 这一段继续实现，围绕 `if (const auto *Call = dyn_cast<CallExpr>(AllocExpr)) {` 展开声明或语句。

### Lines 76-84
```cpp
  76 |       const auto *CtrE = New->getConstructExpr();
  77 |       if (!CtrE || !CtrE->getArg(CtrE->getNumArgs() - 1)
  78 |                         ->getType()
  79 |                         ->isIntegralOrEnumerationType())
  80 |         return;
  81 |       CallName = "operator new";
  82 |     }
  83 |   }
  84 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const auto *CtrE = New->getConstructExpr();`.
- CN: 这一段继续实现，围绕 `const auto *CtrE = New->getConstructExpr();` 展开声明或语句。

### Lines 85-91
```cpp
  85 |   const SourceRange OldRParen = SourceRange(PtrArith->getLHS()->getEndLoc());
  86 |   const StringRef RParen =
  87 |       Lexer::getSourceText(CharSourceRange::getTokenRange(OldRParen),
  88 |                            *Result.SourceManager, getLangOpts());
  89 |   const SourceLocation NewRParen = Lexer::getLocForEndOfToken(
  90 |       PtrArith->getEndLoc(), 0, *Result.SourceManager, getLangOpts());
  91 | 
```
- EN: Method definitions such as `Lexer::getSourceText` provide the concrete behavior declared elsewhere.
- CN: 诸如 `Lexer::getSourceText` 的方法定义给出了前面声明的具体行为。

### Lines 92-99
```cpp
  92 |   diag(PtrArith->getBeginLoc(),
  93 |        "arithmetic operation is applied to the result of %0() instead of its "
  94 |        "size-like argument")
  95 |       << CallName << FixItHint::CreateRemoval(OldRParen)
  96 |       << FixItHint::CreateInsertion(NewRParen, RParen);
  97 | }
  98 | 
  99 | } // namespace clang::tidy::bugprone
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。
- EN: Method definitions such as `FixItHint::CreateRemoval`, `FixItHint::CreateInsertion` provide the concrete behavior declared elsewhere.
- CN: 诸如 `FixItHint::CreateRemoval`、`FixItHint::CreateInsertion` 的方法定义给出了前面声明的具体行为。

## Key Concepts / 关键概念
- AST matcher DSL / AST 匹配器 DSL
- diagnostics and fix-its / 诊断与修复建议
- namespace scoping / 命名空间作用域

## Dependencies / 依赖关系
- EN: Direct includes: `MisplacedPointerArithmeticInAllocCheck.h`, `clang/AST/ASTContext.h`, `clang/ASTMatchers/ASTMatchFinder.h`, `clang/Lex/Lexer.h`.
- CN: 直接包含依赖: `MisplacedPointerArithmeticInAllocCheck.h`、`clang/AST/ASTContext.h`、`clang/ASTMatchers/ASTMatchFinder.h`、`clang/Lex/Lexer.h`。
- EN: Namespace context: `clang::tidy::bugprone`.
- CN: 命名空间上下文: `clang::tidy::bugprone`。
