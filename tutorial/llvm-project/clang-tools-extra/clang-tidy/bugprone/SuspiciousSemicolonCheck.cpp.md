# SuspiciousSemicolonCheck.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `clang-tools-extra/clang-tidy/bugprone/SuspiciousSemicolonCheck.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `SuspiciousSemicolonCheck` by registering AST matchers and producing diagnostics/fix-its.
- 用途 (CN): 通过注册 AST 匹配器并生成诊断/修复建议来实现 `SuspiciousSemicolonCheck`。

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
   9 | #include "SuspiciousSemicolonCheck.h"
  10 | #include "../utils/LexerUtils.h"
  11 | #include "clang/AST/ASTContext.h"
  12 | #include "clang/ASTMatchers/ASTMatchFinder.h"
  13 | 
```
- EN: The section imports dependencies such as `SuspiciousSemicolonCheck.h`, `../utils/LexerUtils.h`, `clang/AST/ASTContext.h`, `clang/ASTMatchers/ASTMatchFinder.h` needed by this file.
- CN: 本段引入了 `SuspiciousSemicolonCheck.h`、`../utils/LexerUtils.h`、`clang/AST/ASTContext.h`、`clang/ASTMatchers/ASTMatchFinder.h` 等依赖，供当前文件使用。

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

### Lines 18-27
```cpp
  18 | void SuspiciousSemicolonCheck::registerMatchers(MatchFinder *Finder) {
  19 |   Finder->addMatcher(ifStmt(hasThen(nullStmt().bind("semi")),
  20 |                             unless(hasElse(stmt())), unless(isConstexpr()))
  21 |                          .bind("stmt"),
  22 |                      this);
  23 |   Finder->addMatcher(forStmt(hasBody(nullStmt().bind("semi"))).bind("stmt"),
  24 |                      this);
  25 |   Finder->addMatcher(
  26 |       cxxForRangeStmt(hasBody(nullStmt().bind("semi"))).bind("stmt"), this);
  27 |   Finder->addMatcher(whileStmt(hasBody(nullStmt().bind("semi"))).bind("stmt"),
```
- EN: This block defines AST matcher registration, i.e. the structural patterns the check will look for.
- CN: 这一段定义 AST 匹配器注册，也就是该检查要查找的结构模式。
- EN: Method definitions such as `SuspiciousSemicolonCheck::registerMatchers` provide the concrete behavior declared elsewhere.
- CN: 诸如 `SuspiciousSemicolonCheck::registerMatchers` 的方法定义给出了前面声明的具体行为。

### Lines 28-34
```cpp
  28 |                      this);
  29 | }
  30 | 
  31 | void SuspiciousSemicolonCheck::check(const MatchFinder::MatchResult &Result) {
  32 |   if (Result.Context->getDiagnostics().hasUncompilableErrorOccurred())
  33 |     return;
  34 | 
```
- EN: Method definitions such as `SuspiciousSemicolonCheck::check` provide the concrete behavior declared elsewhere.
- CN: 诸如 `SuspiciousSemicolonCheck::check` 的方法定义给出了前面声明的具体行为。

### Lines 35-40
```cpp
  35 |   const auto *Semicolon = Result.Nodes.getNodeAs<NullStmt>("semi");
  36 |   const SourceLocation LocStart = Semicolon->getBeginLoc();
  37 | 
  38 |   if (LocStart.isMacroID())
  39 |     return;
  40 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const auto *Semicolon = Result.Nodes.getNodeAs<NullStmt>("se`.
- CN: 这一段继续实现，围绕 `const auto *Semicolon = Result.Nodes.getNodeAs<NullStmt>("se` 展开声明或语句。

### Lines 41-44
```cpp
  41 |   ASTContext &Ctxt = *Result.Context;
  42 |   const auto &SM = *Result.SourceManager;
  43 |   const unsigned SemicolonLine = SM.getSpellingLineNumber(LocStart);
  44 | 
```
- EN: This block continues the implementation with declarations or statements centered on `ASTContext &Ctxt = *Result.Context;`.
- CN: 这一段继续实现，围绕 `ASTContext &Ctxt = *Result.Context;` 展开声明或语句。

### Lines 45-53
```cpp
  45 |   const auto *Statement = Result.Nodes.getNodeAs<Stmt>("stmt");
  46 |   const bool IsIfStmt = isa<IfStmt>(Statement);
  47 | 
  48 |   const std::optional<Token> PrevTok = utils::lexer::getPreviousToken(
  49 |       LocStart, Ctxt.getSourceManager(), Ctxt.getLangOpts());
  50 |   if (!PrevTok || (!IsIfStmt && SM.getSpellingLineNumber(
  51 |                                     PrevTok->getLocation()) != SemicolonLine))
  52 |     return;
  53 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const auto *Statement = Result.Nodes.getNodeAs<Stmt>("stmt")`.
- CN: 这一段继续实现，围绕 `const auto *Statement = Result.Nodes.getNodeAs<Stmt>("stmt")` 展开声明或语句。

### Lines 54-63
```cpp
  54 |   const SourceLocation LocEnd = Semicolon->getEndLoc();
  55 |   const FileID FID = SM.getFileID(LocEnd);
  56 |   const llvm::MemoryBufferRef Buffer = SM.getBufferOrFake(FID, LocEnd);
  57 |   Lexer Lexer(SM.getLocForStartOfFile(FID), Ctxt.getLangOpts(),
  58 |               Buffer.getBufferStart(), SM.getCharacterData(LocEnd) + 1,
  59 |               Buffer.getBufferEnd());
  60 |   Token Token;
  61 |   if (Lexer.LexFromRawLexer(Token))
  62 |     return;
  63 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const SourceLocation LocEnd = Semicolon->getEndLoc();`.
- CN: 这一段继续实现，围绕 `const SourceLocation LocEnd = Semicolon->getEndLoc();` 展开声明或语句。

### Lines 64-69
```cpp
  64 |   const unsigned BaseIndent =
  65 |       SM.getSpellingColumnNumber(Statement->getBeginLoc());
  66 |   const unsigned NewTokenIndent =
  67 |       SM.getSpellingColumnNumber(Token.getLocation());
  68 |   const unsigned NewTokenLine = SM.getSpellingLineNumber(Token.getLocation());
  69 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const unsigned BaseIndent =`.
- CN: 这一段继续实现，围绕 `const unsigned BaseIndent =` 展开声明或语句。

### Lines 70-73
```cpp
  70 |   if (!IsIfStmt && NewTokenIndent <= BaseIndent &&
  71 |       Token.getKind() != tok::l_brace && NewTokenLine != SemicolonLine)
  72 |     return;
  73 | 
```
- EN: This block continues the implementation with declarations or statements centered on `if (!IsIfStmt && NewTokenIndent <= BaseIndent &&`.
- CN: 这一段继续实现，围绕 `if (!IsIfStmt && NewTokenIndent <= BaseIndent &&` 展开声明或语句。

### Lines 74-78
```cpp
  74 |   diag(LocStart, "potentially unintended semicolon")
  75 |       << FixItHint::CreateRemoval(SourceRange(LocStart, LocEnd));
  76 | }
  77 | 
  78 | } // namespace clang::tidy::bugprone
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。
- EN: Method definitions such as `FixItHint::CreateRemoval` provide the concrete behavior declared elsewhere.
- CN: 诸如 `FixItHint::CreateRemoval` 的方法定义给出了前面声明的具体行为。

## Key Concepts / 关键概念
- AST matcher DSL / AST 匹配器 DSL
- diagnostics and fix-its / 诊断与修复建议
- namespace scoping / 命名空间作用域

## Dependencies / 依赖关系
- EN: Direct includes: `SuspiciousSemicolonCheck.h`, `../utils/LexerUtils.h`, `clang/AST/ASTContext.h`, `clang/ASTMatchers/ASTMatchFinder.h`.
- CN: 直接包含依赖: `SuspiciousSemicolonCheck.h`、`../utils/LexerUtils.h`、`clang/AST/ASTContext.h`、`clang/ASTMatchers/ASTMatchFinder.h`。
- EN: Namespace context: `clang::tidy::bugprone`.
- CN: 命名空间上下文: `clang::tidy::bugprone`。
