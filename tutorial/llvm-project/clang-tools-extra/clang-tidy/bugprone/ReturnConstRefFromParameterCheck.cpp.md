# ReturnConstRefFromParameterCheck.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `clang-tools-extra/clang-tidy/bugprone/ReturnConstRefFromParameterCheck.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `ReturnConstRefFromParameterCheck` by registering AST matchers and producing diagnostics/fix-its.
- 用途 (CN): 通过注册 AST 匹配器并生成诊断/修复建议来实现 `ReturnConstRefFromParameterCheck`。

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

### Lines 9-14
```cpp
   9 | #include "ReturnConstRefFromParameterCheck.h"
  10 | #include "clang/AST/Attrs.inc"
  11 | #include "clang/AST/Expr.h"
  12 | #include "clang/ASTMatchers/ASTMatchFinder.h"
  13 | #include "clang/ASTMatchers/ASTMatchers.h"
  14 | 
```
- EN: The section imports dependencies such as `ReturnConstRefFromParameterCheck.h`, `clang/AST/Attrs.inc`, `clang/AST/Expr.h`, `clang/ASTMatchers/ASTMatchFinder.h` needed by this file.
- CN: 本段引入了 `ReturnConstRefFromParameterCheck.h`、`clang/AST/Attrs.inc`、`clang/AST/Expr.h`、`clang/ASTMatchers/ASTMatchFinder.h` 等依赖，供当前文件使用。

### Lines 15-18
```cpp
  15 | using namespace clang::ast_matchers;
  16 | 
  17 | namespace clang::tidy::bugprone {
  18 | 
```
- EN: Using-directives shorten qualified names so matcher/transformer code reads more directly.
- CN: using 指令缩短了限定名，使匹配器/变换器代码更直接。
- EN: Namespace scopes such as `clang::tidy::bugprone` place the symbols in their intended subsystem.
- CN: 诸如 `clang::tidy::bugprone` 这样的命名空间将符号放入预期的子系统中。

### Lines 19-24
```cpp
  19 | namespace {
  20 | 
  21 | AST_MATCHER(ParmVarDecl, hasLifetimeBoundAttr) {
  22 |   return Node.hasAttr<LifetimeBoundAttr>();
  23 | }
  24 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 25-34
```cpp
  25 | } // namespace
  26 | 
  27 | void ReturnConstRefFromParameterCheck::registerMatchers(MatchFinder *Finder) {
  28 |   const auto DRef = ignoringParens(
  29 |       declRefExpr(
  30 |           to(parmVarDecl(hasType(hasCanonicalType(
  31 |                              qualType(lValueReferenceType(pointee(
  32 |                                           qualType(isConstQualified()))))
  33 |                                  .bind("type"))),
  34 |                          hasDeclContext(functionDecl(
```
- EN: This block defines AST matcher registration, i.e. the structural patterns the check will look for.
- CN: 这一段定义 AST 匹配器注册，也就是该检查要查找的结构模式。
- EN: Method definitions such as `ReturnConstRefFromParameterCheck::registerMatchers` provide the concrete behavior declared elsewhere.
- CN: 诸如 `ReturnConstRefFromParameterCheck::registerMatchers` 的方法定义给出了前面声明的具体行为。

### Lines 35-41
```cpp
  35 |                              equalsBoundNode("func"),
  36 |                              hasReturnTypeLoc(loc(qualType(
  37 |                                  hasCanonicalType(equalsBoundNode("type"))))))),
  38 |                          unless(hasLifetimeBoundAttr()))
  39 |                  .bind("param")))
  40 |           .bind("dref"));
  41 | 
```
- EN: This block continues the implementation with declarations or statements centered on `equalsBoundNode("func"),`.
- CN: 这一段继续实现，围绕 `equalsBoundNode("func"),` 展开声明或语句。

### Lines 42-50
```cpp
  42 |   Finder->addMatcher(
  43 |       returnStmt(
  44 |           hasAncestor(functionDecl().bind("func")),
  45 |           hasReturnValue(anyOf(
  46 |               DRef, ignoringParens(conditionalOperator(eachOf(
  47 |                         hasTrueExpression(DRef), hasFalseExpression(DRef))))))),
  48 |       this);
  49 | }
  50 | 
```
- EN: This block continues the implementation with declarations or statements centered on `Finder->addMatcher(`.
- CN: 这一段继续实现，围绕 `Finder->addMatcher(` 展开声明或语句。

### Lines 51-54
```cpp
  51 | static bool isSameTypeIgnoringConst(QualType A, QualType B) {
  52 |   return A.getCanonicalType().withConst() == B.getCanonicalType().withConst();
  53 | }
  54 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 55-59
```cpp
  55 | static bool isSameTypeIgnoringConstRef(QualType A, QualType B) {
  56 |   return isSameTypeIgnoringConst(A.getCanonicalType().getNonReferenceType(),
  57 |                                  B.getCanonicalType().getNonReferenceType());
  58 | }
  59 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 60-69
```cpp
  60 | static bool hasSameParameterTypes(const FunctionDecl &FD, const FunctionDecl &O,
  61 |                                   const ParmVarDecl &PD) {
  62 |   if (FD.getNumParams() != O.getNumParams())
  63 |     return false;
  64 |   for (unsigned I = 0, E = FD.getNumParams(); I < E; ++I) {
  65 |     const ParmVarDecl *DPD = FD.getParamDecl(I);
  66 |     const QualType OPT = O.getParamDecl(I)->getType();
  67 |     if (DPD == &PD) {
  68 |       if (!isa<RValueReferenceType>(OPT) ||
  69 |           !isSameTypeIgnoringConstRef(DPD->getType(), OPT))
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 70-78
```cpp
  70 |         return false;
  71 |     } else {
  72 |       if (!isSameTypeIgnoringConst(DPD->getType(), OPT))
  73 |         return false;
  74 |     }
  75 |   }
  76 |   return true;
  77 | }
  78 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 79-88
```cpp
  79 | static const Decl *findRVRefOverload(const FunctionDecl &FD,
  80 |                                      const ParmVarDecl &PD) {
  81 |   // Actually it would be better to do lookup in caller site.
  82 |   // But in most of cases, overloads of LVRef and RVRef will appear together.
  83 |   // FIXME:
  84 |   // 1. overload in anonymous namespace
  85 |   // 2. forward reference
  86 |   const DeclContext::lookup_result LookupResult =
  87 |       FD.getParent()->lookup(FD.getNameInfo().getName());
  88 |   if (LookupResult.isSingleResult())
```
- EN: This block continues the implementation with declarations or statements centered on `static const Decl *findRVRefOverload(const FunctionDecl &FD,`.
- CN: 这一段继续实现，围绕 `static const Decl *findRVRefOverload(const FunctionDecl &FD,` 展开声明或语句。

### Lines 89-98
```cpp
  89 |     return nullptr;
  90 |   for (const Decl *Overload : LookupResult) {
  91 |     if (Overload == &FD)
  92 |       continue;
  93 |     if (const auto *O = dyn_cast<FunctionDecl>(Overload))
  94 |       if (hasSameParameterTypes(FD, *O, PD))
  95 |         return O;
  96 |   }
  97 |   return nullptr;
  98 | }
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 99-108
```cpp
  99 | 
 100 | void ReturnConstRefFromParameterCheck::check(
 101 |     const MatchFinder::MatchResult &Result) {
 102 |   const auto *FD = Result.Nodes.getNodeAs<FunctionDecl>("func");
 103 |   const auto *PD = Result.Nodes.getNodeAs<ParmVarDecl>("param");
 104 |   const auto *DRef = Result.Nodes.getNodeAs<DeclRefExpr>("dref");
 105 |   const SourceRange Range = DRef->getSourceRange();
 106 |   if (Range.isInvalid())
 107 |     return;
 108 | 
```
- EN: Method definitions such as `ReturnConstRefFromParameterCheck::check` provide the concrete behavior declared elsewhere.
- CN: 诸如 `ReturnConstRefFromParameterCheck::check` 的方法定义给出了前面声明的具体行为。

### Lines 109-118
```cpp
 109 |   if (findRVRefOverload(*FD, *PD) != nullptr)
 110 |     return;
 111 | 
 112 |   diag(Range.getBegin(),
 113 |        "returning a constant reference parameter may cause use-after-free "
 114 |        "when the parameter is constructed from a temporary")
 115 |       << Range;
 116 | }
 117 | 
 118 | } // namespace clang::tidy::bugprone
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

## Key Concepts / 关键概念
- AST matcher DSL / AST 匹配器 DSL
- diagnostics and fix-its / 诊断与修复建议
- namespace scoping / 命名空间作用域

## Dependencies / 依赖关系
- EN: Direct includes: `ReturnConstRefFromParameterCheck.h`, `clang/AST/Attrs.inc`, `clang/AST/Expr.h`, `clang/ASTMatchers/ASTMatchFinder.h`, `clang/ASTMatchers/ASTMatchers.h`.
- CN: 直接包含依赖: `ReturnConstRefFromParameterCheck.h`、`clang/AST/Attrs.inc`、`clang/AST/Expr.h`、`clang/ASTMatchers/ASTMatchFinder.h`、`clang/ASTMatchers/ASTMatchers.h`。
- EN: Namespace context: `clang::tidy::bugprone`.
- CN: 命名空间上下文: `clang::tidy::bugprone`。
