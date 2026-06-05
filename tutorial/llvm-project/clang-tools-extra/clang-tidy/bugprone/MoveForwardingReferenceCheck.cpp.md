# MoveForwardingReferenceCheck.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `clang-tools-extra/clang-tidy/bugprone/MoveForwardingReferenceCheck.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `Lexer` by registering AST matchers and producing diagnostics/fix-its.
- 用途 (CN): 通过注册 AST 匹配器并生成诊断/修复建议来实现 `Lexer`。

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
   9 | #include "MoveForwardingReferenceCheck.h"
  10 | #include "clang/Lex/Lexer.h"
  11 | 
  12 | using namespace clang::ast_matchers;
  13 | 
```
- EN: The section imports dependencies such as `MoveForwardingReferenceCheck.h`, `clang/Lex/Lexer.h` needed by this file.
- CN: 本段引入了 `MoveForwardingReferenceCheck.h`、`clang/Lex/Lexer.h` 等依赖，供当前文件使用。
- EN: Using-directives shorten qualified names so matcher/transformer code reads more directly.
- CN: using 指令缩短了限定名，使匹配器/变换器代码更直接。

### Lines 14-17
```cpp
  14 | namespace clang::tidy::bugprone {
  15 | 
  16 | namespace {
  17 | 
```
- EN: Namespace scopes such as `clang::tidy::bugprone` place the symbols in their intended subsystem.
- CN: 诸如 `clang::tidy::bugprone` 这样的命名空间将符号放入预期的子系统中。

### Lines 18-21
```cpp
  18 | AST_MATCHER(DeclRefExpr, refersToEnclosingVariableOrCapture) {
  19 |   return Node.refersToEnclosingVariableOrCapture();
  20 | }
  21 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 22-31
```cpp
  22 | } // namespace
  23 | 
  24 | static void replaceMoveWithForward(const UnresolvedLookupExpr *Callee,
  25 |                                    const ParmVarDecl *ParmVar,
  26 |                                    const TemplateTypeParmDecl *TypeParmDecl,
  27 |                                    DiagnosticBuilder &Diag,
  28 |                                    const ASTContext &Context) {
  29 |   const SourceManager &SM = Context.getSourceManager();
  30 |   const LangOptions &LangOpts = Context.getLangOpts();
  31 | 
```
- EN: This block continues the implementation with declarations or statements centered on `} // namespace`.
- CN: 这一段继续实现，围绕 `} // namespace` 展开声明或语句。

### Lines 32-36
```cpp
  32 |   const CharSourceRange CallRange =
  33 |       Lexer::makeFileCharRange(CharSourceRange::getTokenRange(
  34 |                                    Callee->getBeginLoc(), Callee->getEndLoc()),
  35 |                                SM, LangOpts);
  36 | 
```
- EN: Method definitions such as `Lexer::makeFileCharRange` provide the concrete behavior declared elsewhere.
- CN: 诸如 `Lexer::makeFileCharRange` 的方法定义给出了前面声明的具体行为。

### Lines 37-42
```cpp
  37 |   if (CallRange.isValid()) {
  38 |     const std::string TypeName =
  39 |         (TypeParmDecl->getIdentifier() && !TypeParmDecl->isImplicit())
  40 |             ? TypeParmDecl->getName().str()
  41 |             : (llvm::Twine("decltype(") + ParmVar->getName() + ")").str();
  42 | 
```
- EN: This block continues the implementation with declarations or statements centered on `if (CallRange.isValid()) {`.
- CN: 这一段继续实现，围绕 `if (CallRange.isValid()) {` 展开声明或语句。

### Lines 43-52
```cpp
  43 |     const std::string ForwardName =
  44 |         (llvm::Twine("forward<") + TypeName + ">").str();
  45 | 
  46 |     // Create a replacement only if we see a "standard" way of calling
  47 |     // std::move(). This will hopefully prevent erroneous replacements if the
  48 |     // code does unusual things (e.g. create an alias for std::move() in
  49 |     // another namespace).
  50 |     const NestedNameSpecifier NNS = Callee->getQualifier();
  51 |     switch (NNS.getKind()) {
  52 |     case NestedNameSpecifier::Kind::Null:
```
- EN: This block continues the implementation with declarations or statements centered on `const std::string ForwardName =`.
- CN: 这一段继续实现，围绕 `const std::string ForwardName =` 展开声明或语句。

### Lines 53-62
```cpp
  53 |       // Called as "move" (i.e. presumably the code had a "using std::move;").
  54 |       // We still conservatively put a "std::" in front of the forward because
  55 |       // we don't know whether the code also had a "using std::forward;".
  56 |       Diag << FixItHint::CreateReplacement(CallRange, "std::" + ForwardName);
  57 |       break;
  58 |     case NestedNameSpecifier::Kind::Namespace: {
  59 |       auto [Namespace, Prefix] = NNS.getAsNamespaceAndPrefix();
  60 |       if (Namespace->getName() == "std") {
  61 |         if (!Prefix) {
  62 |           // Called as "std::move".
```
- EN: Method definitions such as `FixItHint::CreateReplacement` provide the concrete behavior declared elsewhere.
- CN: 诸如 `FixItHint::CreateReplacement` 的方法定义给出了前面声明的具体行为。

### Lines 63-72
```cpp
  63 |           Diag << FixItHint::CreateReplacement(CallRange,
  64 |                                                "std::" + ForwardName);
  65 |         } else if (Prefix.getKind() == NestedNameSpecifier::Kind::Global) {
  66 |           // Called as "::std::move".
  67 |           Diag << FixItHint::CreateReplacement(CallRange,
  68 |                                                "::std::" + ForwardName);
  69 |         }
  70 |       }
  71 |       break;
  72 |     }
```
- EN: Method definitions such as `FixItHint::CreateReplacement` provide the concrete behavior declared elsewhere.
- CN: 诸如 `FixItHint::CreateReplacement` 的方法定义给出了前面声明的具体行为。

### Lines 73-78
```cpp
  73 |     default:
  74 |       return;
  75 |     }
  76 |   }
  77 | }
  78 | 
```
- EN: This block continues the implementation with declarations or statements centered on `default:`.
- CN: 这一段继续实现，围绕 `default:` 展开声明或语句。

### Lines 79-88
```cpp
  79 | void MoveForwardingReferenceCheck::registerMatchers(MatchFinder *Finder) {
  80 |   // Matches a ParmVarDecl for a forwarding reference, i.e. a non-const rvalue
  81 |   // reference of a function template parameter type.
  82 |   auto ForwardingReferenceParmMatcher =
  83 |       parmVarDecl(
  84 |           hasType(qualType(rValueReferenceType(),
  85 |                            references(templateTypeParmType(hasDeclaration(
  86 |                                templateTypeParmDecl().bind("type-parm-decl")))),
  87 |                            unless(references(qualType(isConstQualified()))))))
  88 |           .bind("parm-var");
```
- EN: This block defines AST matcher registration, i.e. the structural patterns the check will look for.
- CN: 这一段定义 AST 匹配器注册，也就是该检查要查找的结构模式。
- EN: Method definitions such as `MoveForwardingReferenceCheck::registerMatchers` provide the concrete behavior declared elsewhere.
- CN: 诸如 `MoveForwardingReferenceCheck::registerMatchers` 的方法定义给出了前面声明的具体行为。

### Lines 89-98
```cpp
  89 | 
  90 |   Finder->addMatcher(
  91 |       callExpr(
  92 |           callee(unresolvedLookupExpr(
  93 |                      hasAnyDeclaration(
  94 |                          namedDecl(hasUnderlyingDecl(hasName("::std::move")))))
  95 |                      .bind("lookup")),
  96 |           argumentCountIs(1),
  97 |           hasArgument(0, ignoringParenImpCasts(declRefExpr(
  98 |                              to(ForwardingReferenceParmMatcher),
```
- EN: This block continues the implementation with declarations or statements centered on `Finder->addMatcher(`.
- CN: 这一段继续实现，围绕 `Finder->addMatcher(` 展开声明或语句。

### Lines 99-103
```cpp
  99 |                              unless(refersToEnclosingVariableOrCapture())))))
 100 |           .bind("call-move"),
 101 |       this);
 102 | }
 103 | 
```
- EN: This block continues the implementation with declarations or statements centered on `unless(refersToEnclosingVariableOrCapture())))))`.
- CN: 这一段继续实现，围绕 `unless(refersToEnclosingVariableOrCapture())))))` 展开声明或语句。

### Lines 104-112
```cpp
 104 | void MoveForwardingReferenceCheck::check(
 105 |     const MatchFinder::MatchResult &Result) {
 106 |   const auto *CallMove = Result.Nodes.getNodeAs<CallExpr>("call-move");
 107 |   const auto *UnresolvedLookup =
 108 |       Result.Nodes.getNodeAs<UnresolvedLookupExpr>("lookup");
 109 |   const auto *ParmVar = Result.Nodes.getNodeAs<ParmVarDecl>("parm-var");
 110 |   const auto *TypeParmDecl =
 111 |       Result.Nodes.getNodeAs<TemplateTypeParmDecl>("type-parm-decl");
 112 | 
```
- EN: Method definitions such as `MoveForwardingReferenceCheck::check` provide the concrete behavior declared elsewhere.
- CN: 诸如 `MoveForwardingReferenceCheck::check` 的方法定义给出了前面声明的具体行为。

### Lines 113-122
```cpp
 113 |   // Get the FunctionDecl and FunctionTemplateDecl containing the function
 114 |   // parameter.
 115 |   const auto *FuncForParam = dyn_cast<FunctionDecl>(ParmVar->getDeclContext());
 116 |   if (!FuncForParam)
 117 |     return;
 118 |   const FunctionTemplateDecl *FuncTemplate =
 119 |       FuncForParam->getDescribedFunctionTemplate();
 120 |   if (!FuncTemplate)
 121 |     return;
 122 | 
```
- EN: This block continues the implementation with declarations or statements centered on `// Get the FunctionDecl and FunctionTemplateDecl containing `.
- CN: 这一段继续实现，围绕 `// Get the FunctionDecl and FunctionTemplateDecl containing ` 展开声明或语句。

### Lines 123-129
```cpp
 123 |   // Check that the template type parameter belongs to the same function
 124 |   // template as the function parameter of that type. (This implies that type
 125 |   // deduction will happen on the type.)
 126 |   const TemplateParameterList *Params = FuncTemplate->getTemplateParameters();
 127 |   if (!llvm::is_contained(*Params, TypeParmDecl))
 128 |     return;
 129 | 
```
- EN: This block continues the implementation with declarations or statements centered on `// Check that the template type parameter belongs to the sam`.
- CN: 这一段继续实现，围绕 `// Check that the template type parameter belongs to the sam` 展开声明或语句。

### Lines 130-134
```cpp
 130 |   auto Diag = diag(CallMove->getExprLoc(),
 131 |                    "forwarding reference passed to std::move(), which may "
 132 |                    "unexpectedly cause lvalues to be moved; use "
 133 |                    "std::forward() instead");
 134 | 
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

### Lines 135-139
```cpp
 135 |   replaceMoveWithForward(UnresolvedLookup, ParmVar, TypeParmDecl, Diag,
 136 |                          *Result.Context);
 137 | }
 138 | 
 139 | } // namespace clang::tidy::bugprone
```
- EN: This block continues the implementation with declarations or statements centered on `replaceMoveWithForward(UnresolvedLookup, ParmVar, TypeParmDe`.
- CN: 这一段继续实现，围绕 `replaceMoveWithForward(UnresolvedLookup, ParmVar, TypeParmDe` 展开声明或语句。

## Key Concepts / 关键概念
- AST matcher DSL / AST 匹配器 DSL
- diagnostics and fix-its / 诊断与修复建议
- namespace scoping / 命名空间作用域

## Dependencies / 依赖关系
- EN: Direct includes: `MoveForwardingReferenceCheck.h`, `clang/Lex/Lexer.h`.
- CN: 直接包含依赖: `MoveForwardingReferenceCheck.h`、`clang/Lex/Lexer.h`。
- EN: Namespace context: `clang::tidy::bugprone`.
- CN: 命名空间上下文: `clang::tidy::bugprone`。
