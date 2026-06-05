# StringFindStartswithCheck.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `clang-tools-extra/clang-tidy/abseil/StringFindStartswithCheck.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `StringFindStartswithCheck` by registering AST matchers and producing diagnostics/fix-its.
- 用途 (CN): 通过注册 AST 匹配器并生成诊断/修复建议来实现 `StringFindStartswithCheck`。

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

### Lines 9-17
```cpp
   9 | #include "StringFindStartswithCheck.h"
  10 | 
  11 | #include "../utils/OptionsUtils.h"
  12 | #include "clang/AST/ASTContext.h"
  13 | #include "clang/ASTMatchers/ASTMatchers.h"
  14 | #include "clang/Frontend/CompilerInstance.h"
  15 | #include "clang/Lex/Lexer.h"
  16 | #include "clang/Lex/Preprocessor.h"
  17 | 
```
- EN: The section imports dependencies such as `StringFindStartswithCheck.h`, `../utils/OptionsUtils.h`, `clang/AST/ASTContext.h`, `clang/ASTMatchers/ASTMatchers.h` needed by this file.
- CN: 本段引入了 `StringFindStartswithCheck.h`、`../utils/OptionsUtils.h`、`clang/AST/ASTContext.h`、`clang/ASTMatchers/ASTMatchers.h` 等依赖，供当前文件使用。

### Lines 18-21
```cpp
  18 | using namespace clang::ast_matchers;
  19 | 
  20 | namespace clang::tidy::abseil {
  21 | 
```
- EN: Using-directives shorten qualified names so matcher/transformer code reads more directly.
- CN: using 指令缩短了限定名，使匹配器/变换器代码更直接。
- EN: Namespace scopes such as `clang::tidy::abseil` place the symbols in their intended subsystem.
- CN: 诸如 `clang::tidy::abseil` 这样的命名空间将符号放入预期的子系统中。

### Lines 22-31
```cpp
  22 | const auto DefaultStringLikeClasses =
  23 |     "::std::basic_string;::std::basic_string_view";
  24 | 
  25 | StringFindStartswithCheck::StringFindStartswithCheck(StringRef Name,
  26 |                                                      ClangTidyContext *Context)
  27 |     : ClangTidyCheck(Name, Context),
  28 |       StringLikeClasses(utils::options::parseStringList(
  29 |           Options.get("StringLikeClasses", DefaultStringLikeClasses))),
  30 |       IncludeInserter(Options.getLocalOrGlobal("IncludeStyle",
  31 |                                                utils::IncludeSorter::IS_LLVM),
```
- EN: Method definitions such as `StringFindStartswithCheck::StringFindStartswithCheck` provide the concrete behavior declared elsewhere.
- CN: 诸如 `StringFindStartswithCheck::StringFindStartswithCheck` 的方法定义给出了前面声明的具体行为。

### Lines 32-35
```cpp
  32 |                       areDiagsSelfContained()),
  33 |       AbseilStringsMatchHeader(
  34 |           Options.get("AbseilStringsMatchHeader", "absl/strings/match.h")) {}
  35 | 
```
- EN: This block continues the implementation with declarations or statements centered on `areDiagsSelfContained()),`.
- CN: 这一段继续实现，围绕 `areDiagsSelfContained()),` 展开声明或语句。

### Lines 36-41
```cpp
  36 | void StringFindStartswithCheck::registerMatchers(MatchFinder *Finder) {
  37 |   auto ZeroLiteral = integerLiteral(equals(0));
  38 |   auto StringClassMatcher = cxxRecordDecl(hasAnyName(StringLikeClasses));
  39 |   auto StringType = hasUnqualifiedDesugaredType(
  40 |       recordType(hasDeclaration(StringClassMatcher)));
  41 | 
```
- EN: This block defines AST matcher registration, i.e. the structural patterns the check will look for.
- CN: 这一段定义 AST 匹配器注册，也就是该检查要查找的结构模式。
- EN: Method definitions such as `StringFindStartswithCheck::registerMatchers` provide the concrete behavior declared elsewhere.
- CN: 诸如 `StringFindStartswithCheck::registerMatchers` 的方法定义给出了前面声明的具体行为。

### Lines 42-50
```cpp
  42 |   auto StringFind = cxxMemberCallExpr(
  43 |       // .find()-call on a string...
  44 |       callee(cxxMethodDecl(hasName("find")).bind("findfun")),
  45 |       on(hasType(StringType)),
  46 |       // ... with some search expression ...
  47 |       hasArgument(0, expr().bind("needle")),
  48 |       // ... and either "0" as second argument or the default argument (also 0).
  49 |       anyOf(hasArgument(1, ZeroLiteral), hasArgument(1, cxxDefaultArgExpr())));
  50 | 
```
- EN: This block continues the implementation with declarations or statements centered on `auto StringFind = cxxMemberCallExpr(`.
- CN: 这一段继续实现，围绕 `auto StringFind = cxxMemberCallExpr(` 展开声明或语句。

### Lines 51-59
```cpp
  51 |   Finder->addMatcher(
  52 |       // Match [=!]= with a zero on one side and a string.find on the other.
  53 |       binaryOperator(
  54 |           hasAnyOperatorName("==", "!="),
  55 |           hasOperands(ignoringParenImpCasts(ZeroLiteral),
  56 |                       ignoringParenImpCasts(StringFind.bind("findexpr"))))
  57 |           .bind("expr"),
  58 |       this);
  59 | 
```
- EN: This block continues the implementation with declarations or statements centered on `Finder->addMatcher(`.
- CN: 这一段继续实现，围绕 `Finder->addMatcher(` 展开声明或语句。

### Lines 60-68
```cpp
  60 |   auto StringRFind = cxxMemberCallExpr(
  61 |       // .rfind()-call on a string...
  62 |       callee(cxxMethodDecl(hasName("rfind")).bind("findfun")),
  63 |       on(hasType(StringType)),
  64 |       // ... with some search expression ...
  65 |       hasArgument(0, expr().bind("needle")),
  66 |       // ... and "0" as second argument.
  67 |       hasArgument(1, ZeroLiteral));
  68 | 
```
- EN: This block continues the implementation with declarations or statements centered on `auto StringRFind = cxxMemberCallExpr(`.
- CN: 这一段继续实现，围绕 `auto StringRFind = cxxMemberCallExpr(` 展开声明或语句。

### Lines 69-78
```cpp
  69 |   Finder->addMatcher(
  70 |       // Match [=!]= with either a zero or npos on one side and a string.rfind
  71 |       // on the other.
  72 |       binaryOperator(
  73 |           hasAnyOperatorName("==", "!="),
  74 |           hasOperands(ignoringParenImpCasts(ZeroLiteral),
  75 |                       ignoringParenImpCasts(StringRFind.bind("findexpr"))))
  76 |           .bind("expr"),
  77 |       this);
  78 | }
```
- EN: This block continues the implementation with declarations or statements centered on `Finder->addMatcher(`.
- CN: 这一段继续实现，围绕 `Finder->addMatcher(` 展开声明或语句。

### Lines 79-83
```cpp
  79 | 
  80 | void StringFindStartswithCheck::check(const MatchFinder::MatchResult &Result) {
  81 |   const ASTContext &Context = *Result.Context;
  82 |   const SourceManager &Source = Context.getSourceManager();
  83 | 
```
- EN: Method definitions such as `StringFindStartswithCheck::check` provide the concrete behavior declared elsewhere.
- CN: 诸如 `StringFindStartswithCheck::check` 的方法定义给出了前面声明的具体行为。

### Lines 84-93
```cpp
  84 |   // Extract matching (sub)expressions
  85 |   const auto *ComparisonExpr = Result.Nodes.getNodeAs<BinaryOperator>("expr");
  86 |   assert(ComparisonExpr != nullptr);
  87 |   const auto *Needle = Result.Nodes.getNodeAs<Expr>("needle");
  88 |   assert(Needle != nullptr);
  89 |   const Expr *Haystack = Result.Nodes.getNodeAs<CXXMemberCallExpr>("findexpr")
  90 |                              ->getImplicitObjectArgument();
  91 |   assert(Haystack != nullptr);
  92 |   const auto *FindFun = Result.Nodes.getNodeAs<CXXMethodDecl>("findfun");
  93 |   assert(FindFun != nullptr);
```
- EN: This block continues the implementation with declarations or statements centered on `// Extract matching (sub)expressions`.
- CN: 这一段继续实现，围绕 `// Extract matching (sub)expressions` 展开声明或语句。

### Lines 94-99
```cpp
  94 | 
  95 |   const bool Rev = FindFun->getName().contains("rfind");
  96 | 
  97 |   if (ComparisonExpr->getBeginLoc().isMacroID())
  98 |     return;
  99 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const bool Rev = FindFun->getName().contains("rfind");`.
- CN: 这一段继续实现，围绕 `const bool Rev = FindFun->getName().contains("rfind");` 展开声明或语句。

### Lines 100-108
```cpp
 100 |   // Get the source code blocks (as characters) for both the string object
 101 |   // and the search expression
 102 |   const StringRef NeedleExprCode = Lexer::getSourceText(
 103 |       CharSourceRange::getTokenRange(Needle->getSourceRange()), Source,
 104 |       Context.getLangOpts());
 105 |   const StringRef HaystackExprCode = Lexer::getSourceText(
 106 |       CharSourceRange::getTokenRange(Haystack->getSourceRange()), Source,
 107 |       Context.getLangOpts());
 108 | 
```
- EN: Method definitions such as `CharSourceRange::getTokenRange` provide the concrete behavior declared elsewhere.
- CN: 诸如 `CharSourceRange::getTokenRange` 的方法定义给出了前面声明的具体行为。

### Lines 109-118
```cpp
 109 |   // Create the StartsWith string, negating if comparison was "!=".
 110 |   const bool Neg = ComparisonExpr->getOpcode() == BO_NE;
 111 | 
 112 |   // Create the warning message and a FixIt hint replacing the original expr.
 113 |   auto Diagnostic =
 114 |       diag(ComparisonExpr->getBeginLoc(),
 115 |            "use %select{absl::StartsWith|!absl::StartsWith}0 "
 116 |            "instead of %select{find()|rfind()}1 %select{==|!=}0 0")
 117 |       << Neg << Rev;
 118 | 
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

### Lines 119-124
```cpp
 119 |   Diagnostic << FixItHint::CreateReplacement(
 120 |       ComparisonExpr->getSourceRange(),
 121 |       ((Neg ? "!absl::StartsWith(" : "absl::StartsWith(") + HaystackExprCode +
 122 |        ", " + NeedleExprCode + ")")
 123 |           .str());
 124 | 
```
- EN: Method definitions such as `FixItHint::CreateReplacement` provide the concrete behavior declared elsewhere.
- CN: 诸如 `FixItHint::CreateReplacement` 的方法定义给出了前面声明的具体行为。

### Lines 125-131
```cpp
 125 |   // Create a preprocessor #include FixIt hint (createIncludeInsertion checks
 126 |   // whether this already exists).
 127 |   Diagnostic << IncludeInserter.createIncludeInsertion(
 128 |       Source.getFileID(ComparisonExpr->getBeginLoc()),
 129 |       AbseilStringsMatchHeader);
 130 | }
 131 | 
```
- EN: This block continues the implementation with declarations or statements centered on `// Create a preprocessor #include FixIt hint (createIncludeI`.
- CN: 这一段继续实现，围绕 `// Create a preprocessor #include FixIt hint (createIncludeI` 展开声明或语句。

### Lines 132-136
```cpp
 132 | void StringFindStartswithCheck::registerPPCallbacks(
 133 |     const SourceManager &SM, Preprocessor *PP, Preprocessor *ModuleExpanderPP) {
 134 |   IncludeInserter.registerPreprocessor(PP);
 135 | }
 136 | 
```
- EN: Method definitions such as `StringFindStartswithCheck::registerPPCallbacks` provide the concrete behavior declared elsewhere.
- CN: 诸如 `StringFindStartswithCheck::registerPPCallbacks` 的方法定义给出了前面声明的具体行为。

### Lines 137-145
```cpp
 137 | void StringFindStartswithCheck::storeOptions(
 138 |     ClangTidyOptions::OptionMap &Opts) {
 139 |   Options.store(Opts, "StringLikeClasses",
 140 |                 utils::options::serializeStringList(StringLikeClasses));
 141 |   Options.store(Opts, "IncludeStyle", IncludeInserter.getStyle());
 142 |   Options.store(Opts, "AbseilStringsMatchHeader", AbseilStringsMatchHeader);
 143 | }
 144 | 
 145 | } // namespace clang::tidy::abseil
```
- EN: Method definitions such as `StringFindStartswithCheck::storeOptions` provide the concrete behavior declared elsewhere.
- CN: 诸如 `StringFindStartswithCheck::storeOptions` 的方法定义给出了前面声明的具体行为。

## Key Concepts / 关键概念
- clang-tidy check lifecycle / clang-tidy 检查生命周期
- AST matcher DSL / AST 匹配器 DSL
- diagnostics and fix-its / 诊断与修复建议
- namespace scoping / 命名空间作用域

## Dependencies / 依赖关系
- EN: Direct includes: `StringFindStartswithCheck.h`, `../utils/OptionsUtils.h`, `clang/AST/ASTContext.h`, `clang/ASTMatchers/ASTMatchers.h`, `clang/Frontend/CompilerInstance.h`, `clang/Lex/Lexer.h`, `clang/Lex/Preprocessor.h`.
- CN: 直接包含依赖: `StringFindStartswithCheck.h`、`../utils/OptionsUtils.h`、`clang/AST/ASTContext.h`、`clang/ASTMatchers/ASTMatchers.h`、`clang/Frontend/CompilerInstance.h`、`clang/Lex/Lexer.h`、`clang/Lex/Preprocessor.h`。
- EN: Namespace context: `clang::tidy::abseil`.
- CN: 命名空间上下文: `clang::tidy::abseil`。
