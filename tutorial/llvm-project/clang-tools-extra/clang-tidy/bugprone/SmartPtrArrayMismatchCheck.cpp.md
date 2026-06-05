# SmartPtrArrayMismatchCheck.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `clang-tools-extra/clang-tidy/bugprone/SmartPtrArrayMismatchCheck.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `SmartPtrArrayMismatchCheck` by registering AST matchers and producing diagnostics/fix-its.
- 用途 (CN): 通过注册 AST 匹配器并生成诊断/修复建议来实现 `SmartPtrArrayMismatchCheck`。

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
   9 | #include "SmartPtrArrayMismatchCheck.h"
  10 | #include "../utils/ASTUtils.h"
  11 | #include "clang/ASTMatchers/ASTMatchFinder.h"
  12 | #include "clang/Lex/Lexer.h"
  13 | 
```
- EN: The section imports dependencies such as `SmartPtrArrayMismatchCheck.h`, `../utils/ASTUtils.h`, `clang/ASTMatchers/ASTMatchFinder.h`, `clang/Lex/Lexer.h` needed by this file.
- CN: 本段引入了 `SmartPtrArrayMismatchCheck.h`、`../utils/ASTUtils.h`、`clang/ASTMatchers/ASTMatchFinder.h`、`clang/Lex/Lexer.h` 等依赖，供当前文件使用。

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

### Lines 18-21
```cpp
  18 | static constexpr char ConstructExprN[] = "found_construct_expr";
  19 | static constexpr char NewExprN[] = "found_new_expr";
  20 | static constexpr char ConstructorN[] = "found_constructor";
  21 | 
```
- EN: This block continues the implementation with declarations or statements centered on `static constexpr char ConstructExprN[] = "found_construct_ex`.
- CN: 这一段继续实现，围绕 `static constexpr char ConstructExprN[] = "found_construct_ex` 展开声明或语句。

### Lines 22-30
```cpp
  22 | static bool isInSingleDeclStmt(const DeclaratorDecl *D) {
  23 |   const DynTypedNodeList Parents =
  24 |       D->getASTContext().getParentMapContext().getParents(*D);
  25 |   for (const DynTypedNode &PNode : Parents)
  26 |     if (const auto *PDecl = PNode.get<DeclStmt>())
  27 |       return PDecl->isSingleDecl();
  28 |   return false;
  29 | }
  30 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 31-40
```cpp
  31 | static const DeclaratorDecl *
  32 | getConstructedVarOrField(const Expr *FoundConstructExpr, ASTContext &Ctx) {
  33 |   const DynTypedNodeList ConstructParents =
  34 |       Ctx.getParentMapContext().getParents(*FoundConstructExpr);
  35 |   if (ConstructParents.size() != 1)
  36 |     return nullptr;
  37 |   const auto *ParentDecl = ConstructParents.begin()->get<DeclaratorDecl>();
  38 |   if (isa_and_nonnull<VarDecl, FieldDecl>(ParentDecl))
  39 |     return ParentDecl;
  40 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 41-45
```cpp
  41 |   return nullptr;
  42 | }
  43 | 
  44 | const char SmartPtrArrayMismatchCheck::PointerTypeN[] = "pointer_type";
  45 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 46-49
```cpp
  46 | SmartPtrArrayMismatchCheck::SmartPtrArrayMismatchCheck(
  47 |     StringRef Name, ClangTidyContext *Context, StringRef SmartPointerName)
  48 |     : ClangTidyCheck(Name, Context), SmartPointerName(SmartPointerName) {}
  49 | 
```
- EN: This block continues the implementation with declarations or statements centered on `SmartPtrArrayMismatchCheck::SmartPtrArrayMismatchCheck(`.
- CN: 这一段继续实现，围绕 `SmartPtrArrayMismatchCheck::SmartPtrArrayMismatchCheck(` 展开声明或语句。

### Lines 50-59
```cpp
  50 | void SmartPtrArrayMismatchCheck::storeOptions(
  51 |     ClangTidyOptions::OptionMap &Opts) {}
  52 | 
  53 | void SmartPtrArrayMismatchCheck::registerMatchers(MatchFinder *Finder) {
  54 |   // For both shared and unique pointers, we need to find constructor with
  55 |   // exactly one parameter that has the pointer type. Other constructors are
  56 |   // not applicable for this check.
  57 |   auto FindConstructor =
  58 |       cxxConstructorDecl(ofClass(getSmartPointerClassMatcher()),
  59 |                          parameterCountIs(1), isExplicit())
```
- EN: This block defines AST matcher registration, i.e. the structural patterns the check will look for.
- CN: 这一段定义 AST 匹配器注册，也就是该检查要查找的结构模式。
- EN: Method definitions such as `SmartPtrArrayMismatchCheck::storeOptions`, `SmartPtrArrayMismatchCheck::registerMatchers` provide the concrete behavior declared elsewhere.
- CN: 诸如 `SmartPtrArrayMismatchCheck::storeOptions`、`SmartPtrArrayMismatchCheck::registerMatchers` 的方法定义给出了前面声明的具体行为。

### Lines 60-69
```cpp
  60 |           .bind(ConstructorN);
  61 |   auto FindConstructExpr =
  62 |       cxxConstructExpr(
  63 |           hasDeclaration(FindConstructor), argumentCountIs(1),
  64 |           hasArgument(0,
  65 |                       cxxNewExpr(isArray(),
  66 |                                  hasType(hasCanonicalType(pointerType(
  67 |                                      pointee(equalsBoundNode(PointerTypeN))))))
  68 |                           .bind(NewExprN)))
  69 |           .bind(ConstructExprN);
```
- EN: This block continues the implementation with declarations or statements centered on `.bind(ConstructorN);`.
- CN: 这一段继续实现，围绕 `.bind(ConstructorN);` 展开声明或语句。

### Lines 70-79
```cpp
  70 |   Finder->addMatcher(FindConstructExpr, this);
  71 | }
  72 | 
  73 | void SmartPtrArrayMismatchCheck::check(const MatchFinder::MatchResult &Result) {
  74 |   const auto *FoundNewExpr = Result.Nodes.getNodeAs<CXXNewExpr>(NewExprN);
  75 |   const auto *FoundConstructExpr =
  76 |       Result.Nodes.getNodeAs<CXXConstructExpr>(ConstructExprN);
  77 |   const auto *FoundConstructorDecl =
  78 |       Result.Nodes.getNodeAs<CXXConstructorDecl>(ConstructorN);
  79 | 
```
- EN: Method definitions such as `SmartPtrArrayMismatchCheck::check` provide the concrete behavior declared elsewhere.
- CN: 诸如 `SmartPtrArrayMismatchCheck::check` 的方法定义给出了前面声明的具体行为。

### Lines 80-83
```cpp
  80 |   ASTContext &Ctx = FoundConstructorDecl->getASTContext();
  81 |   const DeclaratorDecl *VarOrField =
  82 |       getConstructedVarOrField(FoundConstructExpr, Ctx);
  83 | 
```
- EN: This block continues the implementation with declarations or statements centered on `ASTContext &Ctx = FoundConstructorDecl->getASTContext();`.
- CN: 这一段继续实现，围绕 `ASTContext &Ctx = FoundConstructorDecl->getASTContext();` 展开声明或语句。

### Lines 84-88
```cpp
  84 |   auto D = diag(FoundNewExpr->getBeginLoc(),
  85 |                 "%0 pointer to non-array is initialized with array")
  86 |            << SmartPointerName;
  87 |   D << FoundNewExpr->getSourceRange();
  88 | 
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

### Lines 89-95
```cpp
  89 |   if (VarOrField) {
  90 |     auto TSTypeLoc = VarOrField->getTypeSourceInfo()
  91 |                          ->getTypeLoc()
  92 |                          .getAsAdjusted<TemplateSpecializationTypeLoc>();
  93 |     assert(TSTypeLoc.getNumArgs() >= 1 &&
  94 |            "Matched type should have at least 1 template argument.");
  95 | 
```
- EN: This block continues the implementation with declarations or statements centered on `if (VarOrField) {`.
- CN: 这一段继续实现，围绕 `if (VarOrField) {` 展开声明或语句。

### Lines 96-101
```cpp
  96 |     const SourceRange TemplateArgumentRange = TSTypeLoc.getArgLoc(0)
  97 |                                                   .getTypeSourceInfo()
  98 |                                                   ->getTypeLoc()
  99 |                                                   .getSourceRange();
 100 |     D << TemplateArgumentRange;
 101 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const SourceRange TemplateArgumentRange = TSTypeLoc.getArgLo`.
- CN: 这一段继续实现，围绕 `const SourceRange TemplateArgumentRange = TSTypeLoc.getArgLo` 展开声明或语句。

### Lines 102-106
```cpp
 102 |     if (isInSingleDeclStmt(VarOrField)) {
 103 |       const SourceManager &SM = Ctx.getSourceManager();
 104 |       if (!utils::rangeCanBeFixed(TemplateArgumentRange, &SM))
 105 |         return;
 106 | 
```
- EN: This block continues the implementation with declarations or statements centered on `if (isInSingleDeclStmt(VarOrField)) {`.
- CN: 这一段继续实现，围绕 `if (isInSingleDeclStmt(VarOrField)) {` 展开声明或语句。

### Lines 107-114
```cpp
 107 |       const SourceLocation InsertLoc = Lexer::getLocForEndOfToken(
 108 |           TemplateArgumentRange.getEnd(), 0, SM, Ctx.getLangOpts());
 109 |       D << FixItHint::CreateInsertion(InsertLoc, "[]");
 110 |     }
 111 |   }
 112 | }
 113 | 
 114 | } // namespace clang::tidy::bugprone
```
- EN: Method definitions such as `FixItHint::CreateInsertion` provide the concrete behavior declared elsewhere.
- CN: 诸如 `FixItHint::CreateInsertion` 的方法定义给出了前面声明的具体行为。

## Key Concepts / 关键概念
- clang-tidy check lifecycle / clang-tidy 检查生命周期
- AST matcher DSL / AST 匹配器 DSL
- diagnostics and fix-its / 诊断与修复建议
- namespace scoping / 命名空间作用域

## Dependencies / 依赖关系
- EN: Direct includes: `SmartPtrArrayMismatchCheck.h`, `../utils/ASTUtils.h`, `clang/ASTMatchers/ASTMatchFinder.h`, `clang/Lex/Lexer.h`.
- CN: 直接包含依赖: `SmartPtrArrayMismatchCheck.h`、`../utils/ASTUtils.h`、`clang/ASTMatchers/ASTMatchFinder.h`、`clang/Lex/Lexer.h`。
- EN: Namespace context: `clang::tidy::bugprone`.
- CN: 命名空间上下文: `clang::tidy::bugprone`。
