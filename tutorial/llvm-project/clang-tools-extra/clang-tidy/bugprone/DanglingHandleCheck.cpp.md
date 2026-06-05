# DanglingHandleCheck.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `clang-tools-extra/clang-tidy/bugprone/DanglingHandleCheck.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `DanglingHandleCheck` by registering AST matchers and producing diagnostics/fix-its.
- 用途 (CN): 通过注册 AST 匹配器并生成诊断/修复建议来实现 `DanglingHandleCheck`。

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
   9 | #include "DanglingHandleCheck.h"
  10 | #include "../utils/OptionsUtils.h"
  11 | #include "clang/AST/ASTContext.h"
  12 | #include "clang/ASTMatchers/ASTMatchFinder.h"
  13 | 
```
- EN: The section imports dependencies such as `DanglingHandleCheck.h`, `../utils/OptionsUtils.h`, `clang/AST/ASTContext.h`, `clang/ASTMatchers/ASTMatchFinder.h` needed by this file.
- CN: 本段引入了 `DanglingHandleCheck.h`、`../utils/OptionsUtils.h`、`clang/AST/ASTContext.h`、`clang/ASTMatchers/ASTMatchFinder.h` 等依赖，供当前文件使用。

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
  18 | static ast_matchers::internal::BindableMatcher<Stmt>
  19 | handleFrom(const ast_matchers::internal::Matcher<RecordDecl> &IsAHandle,
  20 |            const ast_matchers::internal::Matcher<Expr> &Arg) {
  21 |   return expr(
  22 |       anyOf(cxxConstructExpr(hasDeclaration(cxxMethodDecl(ofClass(IsAHandle))),
  23 |                              hasArgument(0, Arg)),
  24 |             cxxMemberCallExpr(hasType(hasUnqualifiedDesugaredType(recordType(
  25 |                                   hasDeclaration(cxxRecordDecl(IsAHandle))))),
  26 |                               callee(memberExpr(member(cxxConversionDecl()))),
  27 |                               on(Arg))));
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 28-37
```cpp
  28 | }
  29 | 
  30 | static ast_matchers::internal::Matcher<Stmt> handleFromTemporaryValue(
  31 |     const ast_matchers::internal::Matcher<RecordDecl> &IsAHandle) {
  32 |   const auto TemporaryExpr = anyOf(
  33 |       cxxBindTemporaryExpr(),
  34 |       cxxFunctionalCastExpr(
  35 |           hasCastKind(CK_ConstructorConversion),
  36 |           hasSourceExpression(ignoringParenImpCasts(cxxBindTemporaryExpr()))));
  37 |   // If a ternary operator returns a temporary value, then both branches hold a
```
- EN: This block continues the implementation with declarations or statements centered on `}`.
- CN: 这一段继续实现，围绕 `}` 展开声明或语句。

### Lines 38-43
```cpp
  38 |   // temporary value. If one of them is not a temporary then it must be copied
  39 |   // into one to satisfy the type of the operator.
  40 |   const auto TemporaryTernary = conditionalOperator(
  41 |       hasTrueExpression(ignoringParenImpCasts(TemporaryExpr)),
  42 |       hasFalseExpression(ignoringParenImpCasts(TemporaryExpr)));
  43 | 
```
- EN: This block continues the implementation with declarations or statements centered on `// temporary value. If one of them is not a temporary then i`.
- CN: 这一段继续实现，围绕 `// temporary value. If one of them is not a temporary then i` 展开声明或语句。

### Lines 44-51
```cpp
  44 |   return handleFrom(IsAHandle, anyOf(TemporaryExpr, TemporaryTernary));
  45 | }
  46 | 
  47 | static ast_matchers::internal::Matcher<RecordDecl> isASequence() {
  48 |   return hasAnyName("::std::deque", "::std::forward_list", "::std::list",
  49 |                     "::std::vector");
  50 | }
  51 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 52-56
```cpp
  52 | static ast_matchers::internal::Matcher<RecordDecl> isASet() {
  53 |   return hasAnyName("::std::set", "::std::multiset", "::std::unordered_set",
  54 |                     "::std::unordered_multiset");
  55 | }
  56 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 57-61
```cpp
  57 | static ast_matchers::internal::Matcher<RecordDecl> isAMap() {
  58 |   return hasAnyName("::std::map", "::std::multimap", "::std::unordered_map",
  59 |                     "::std::unordered_multimap");
  60 | }
  61 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 62-71
```cpp
  62 | static ast_matchers::internal::BindableMatcher<Stmt> makeContainerMatcher(
  63 |     const ast_matchers::internal::Matcher<RecordDecl> &IsAHandle) {
  64 |   // This matcher could be expanded to detect:
  65 |   //  - Constructors: eg. vector<string_view>(3, string("A"));
  66 |   //  - emplace*(): This requires a different logic to determine that
  67 |   //                the conversion will happen inside the container.
  68 |   //  - map's insert: This requires detecting that the pair conversion triggers
  69 |   //                  the bug. A little more complicated than what we have now.
  70 |   return callExpr(
  71 |       hasAnyArgument(
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 72-81
```cpp
  72 |           ignoringParenImpCasts(handleFromTemporaryValue(IsAHandle))),
  73 |       anyOf(
  74 |           // For sequences: assign, push_back, resize.
  75 |           cxxMemberCallExpr(
  76 |               callee(functionDecl(hasAnyName("assign", "push_back", "resize"))),
  77 |               on(expr(hasType(hasUnqualifiedDesugaredType(
  78 |                   recordType(hasDeclaration(recordDecl(isASequence())))))))),
  79 |           // For sequences and sets: insert.
  80 |           cxxMemberCallExpr(callee(functionDecl(hasName("insert"))),
  81 |                             on(expr(hasType(hasUnqualifiedDesugaredType(
```
- EN: This block continues the implementation with declarations or statements centered on `ignoringParenImpCasts(handleFromTemporaryValue(IsAHandle))),`.
- CN: 这一段继续实现，围绕 `ignoringParenImpCasts(handleFromTemporaryValue(IsAHandle))),` 展开声明或语句。

### Lines 82-88
```cpp
  82 |                                 recordType(hasDeclaration(recordDecl(
  83 |                                     anyOf(isASequence(), isASet()))))))))),
  84 |           // For maps: operator[].
  85 |           cxxOperatorCallExpr(callee(cxxMethodDecl(ofClass(isAMap()))),
  86 |                               hasOverloadedOperatorName("[]"))));
  87 | }
  88 | 
```
- EN: This block continues the implementation with declarations or statements centered on `recordType(hasDeclaration(recordDecl(`.
- CN: 这一段继续实现，围绕 `recordType(hasDeclaration(recordDecl(` 展开声明或语句。

### Lines 89-96
```cpp
  89 | DanglingHandleCheck::DanglingHandleCheck(StringRef Name,
  90 |                                          ClangTidyContext *Context)
  91 |     : ClangTidyCheck(Name, Context),
  92 |       HandleClasses(utils::options::parseStringList(Options.get(
  93 |           "HandleClasses", "std::basic_string_view;std::experimental::basic_"
  94 |                            "string_view;std::span"))),
  95 |       IsAHandle(cxxRecordDecl(hasAnyName(HandleClasses)).bind("handle")) {}
  96 | 
```
- EN: This block continues the implementation with declarations or statements centered on `DanglingHandleCheck::DanglingHandleCheck(StringRef Name,`.
- CN: 这一段继续实现，围绕 `DanglingHandleCheck::DanglingHandleCheck(StringRef Name,` 展开声明或语句。

### Lines 97-101
```cpp
  97 | void DanglingHandleCheck::storeOptions(ClangTidyOptions::OptionMap &Opts) {
  98 |   Options.store(Opts, "HandleClasses",
  99 |                 utils::options::serializeStringList(HandleClasses));
 100 | }
 101 | 
```
- EN: Method definitions such as `DanglingHandleCheck::storeOptions` provide the concrete behavior declared elsewhere.
- CN: 诸如 `DanglingHandleCheck::storeOptions` 的方法定义给出了前面声明的具体行为。

### Lines 102-111
```cpp
 102 | void DanglingHandleCheck::registerMatchersForVariables(MatchFinder *Finder) {
 103 |   const auto ConvertedHandle = handleFromTemporaryValue(IsAHandle);
 104 | 
 105 |   // Find 'Handle foo(ReturnsAValue());', 'Handle foo = ReturnsAValue();'
 106 |   Finder->addMatcher(
 107 |       varDecl(hasType(hasUnqualifiedDesugaredType(
 108 |                   recordType(hasDeclaration(cxxRecordDecl(IsAHandle))))),
 109 |               unless(parmVarDecl()),
 110 |               hasInitializer(
 111 |                   exprWithCleanups(ignoringElidableConstructorCall(has(
```
- EN: This block defines AST matcher registration, i.e. the structural patterns the check will look for.
- CN: 这一段定义 AST 匹配器注册，也就是该检查要查找的结构模式。
- EN: Method definitions such as `DanglingHandleCheck::registerMatchersForVariables` provide the concrete behavior declared elsewhere.
- CN: 诸如 `DanglingHandleCheck::registerMatchersForVariables` 的方法定义给出了前面声明的具体行为。

### Lines 112-115
```cpp
 112 |                                        ignoringParenImpCasts(ConvertedHandle))))
 113 |                       .bind("bad_stmt"))),
 114 |       this);
 115 | 
```
- EN: This block continues the implementation with declarations or statements centered on `ignoringParenImpCasts(ConvertedHandle))))`.
- CN: 这一段继续实现，围绕 `ignoringParenImpCasts(ConvertedHandle))))` 展开声明或语句。

### Lines 116-124
```cpp
 116 |   // Find 'foo = ReturnsAValue();  // foo is Handle'
 117 |   Finder->addMatcher(
 118 |       traverse(TK_AsIs,
 119 |                cxxOperatorCallExpr(callee(cxxMethodDecl(ofClass(IsAHandle))),
 120 |                                    hasOverloadedOperatorName("="),
 121 |                                    hasArgument(1, ConvertedHandle))
 122 |                    .bind("bad_stmt")),
 123 |       this);
 124 | 
```
- EN: This block continues the implementation with declarations or statements centered on `// Find 'foo = ReturnsAValue();  // foo is Handle'`.
- CN: 这一段继续实现，围绕 `// Find 'foo = ReturnsAValue();  // foo is Handle'` 展开声明或语句。

### Lines 125-130
```cpp
 125 |   // Container insertions that will dangle.
 126 |   Finder->addMatcher(
 127 |       traverse(TK_AsIs, makeContainerMatcher(IsAHandle).bind("bad_stmt")),
 128 |       this);
 129 | }
 130 | 
```
- EN: This block continues the implementation with declarations or statements centered on `// Container insertions that will dangle.`.
- CN: 这一段继续实现，围绕 `// Container insertions that will dangle.` 展开声明或语句。

### Lines 131-140
```cpp
 131 | void DanglingHandleCheck::registerMatchersForReturn(MatchFinder *Finder) {
 132 |   // Return a local.
 133 |   Finder->addMatcher(
 134 |       traverse(TK_AsIs,
 135 |                returnStmt(
 136 |                    // The AST contains two constructor calls:
 137 |                    //   1. Value to Handle conversion.
 138 |                    //   2. Handle copy construction (elided in C++17+).
 139 |                    // We have to match both.
 140 |                    has(ignoringImplicit(ignoringElidableConstructorCall(
```
- EN: This block defines AST matcher registration, i.e. the structural patterns the check will look for.
- CN: 这一段定义 AST 匹配器注册，也就是该检查要查找的结构模式。
- EN: Method definitions such as `DanglingHandleCheck::registerMatchersForReturn` provide the concrete behavior declared elsewhere.
- CN: 诸如 `DanglingHandleCheck::registerMatchersForReturn` 的方法定义给出了前面声明的具体行为。

### Lines 141-150
```cpp
 141 |                        ignoringImplicit(handleFrom(
 142 |                            IsAHandle,
 143 |                            declRefExpr(to(varDecl(
 144 |                                // Is function scope ...
 145 |                                hasAutomaticStorageDuration(),
 146 |                                // ... and it is a local array or Value.
 147 |                                anyOf(hasType(arrayType()),
 148 |                                      hasType(hasUnqualifiedDesugaredType(
 149 |                                          recordType(hasDeclaration(recordDecl(
 150 |                                              unless(IsAHandle))))))))))))))),
```
- EN: This block continues the implementation with declarations or statements centered on `ignoringImplicit(handleFrom(`.
- CN: 这一段继续实现，围绕 `ignoringImplicit(handleFrom(` 展开声明或语句。

### Lines 151-155
```cpp
 151 |                    // Temporary fix for false positives inside lambdas.
 152 |                    unless(hasAncestor(lambdaExpr())))
 153 |                    .bind("bad_stmt")),
 154 |       this);
 155 | 
```
- EN: This block continues the implementation with declarations or statements centered on `// Temporary fix for false positives inside lambdas.`.
- CN: 这一段继续实现，围绕 `// Temporary fix for false positives inside lambdas.` 展开声明或语句。

### Lines 156-165
```cpp
 156 |   // Return a temporary.
 157 |   Finder->addMatcher(
 158 |       traverse(TK_AsIs,
 159 |                returnStmt(has(exprWithCleanups(ignoringElidableConstructorCall(
 160 |                               has(ignoringParenImpCasts(
 161 |                                   handleFromTemporaryValue(IsAHandle)))))))
 162 |                    .bind("bad_stmt")),
 163 |       this);
 164 | }
 165 | 
```
- EN: This block continues the implementation with declarations or statements centered on `// Return a temporary.`.
- CN: 这一段继续实现，围绕 `// Return a temporary.` 展开声明或语句。

### Lines 166-170
```cpp
 166 | void DanglingHandleCheck::registerMatchers(MatchFinder *Finder) {
 167 |   registerMatchersForVariables(Finder);
 168 |   registerMatchersForReturn(Finder);
 169 | }
 170 | 
```
- EN: This block defines AST matcher registration, i.e. the structural patterns the check will look for.
- CN: 这一段定义 AST 匹配器注册，也就是该检查要查找的结构模式。
- EN: Method definitions such as `DanglingHandleCheck::registerMatchers` provide the concrete behavior declared elsewhere.
- CN: 诸如 `DanglingHandleCheck::registerMatchers` 的方法定义给出了前面声明的具体行为。

### Lines 171-178
```cpp
 171 | void DanglingHandleCheck::check(const MatchFinder::MatchResult &Result) {
 172 |   auto *Handle = Result.Nodes.getNodeAs<CXXRecordDecl>("handle");
 173 |   diag(Result.Nodes.getNodeAs<Stmt>("bad_stmt")->getBeginLoc(),
 174 |        "%0 outlives its value")
 175 |       << Handle->getQualifiedNameAsString();
 176 | }
 177 | 
 178 | } // namespace clang::tidy::bugprone
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。
- EN: Method definitions such as `DanglingHandleCheck::check` provide the concrete behavior declared elsewhere.
- CN: 诸如 `DanglingHandleCheck::check` 的方法定义给出了前面声明的具体行为。

## Key Concepts / 关键概念
- clang-tidy check lifecycle / clang-tidy 检查生命周期
- AST matcher DSL / AST 匹配器 DSL
- diagnostics and fix-its / 诊断与修复建议
- namespace scoping / 命名空间作用域

## Dependencies / 依赖关系
- EN: Direct includes: `DanglingHandleCheck.h`, `../utils/OptionsUtils.h`, `clang/AST/ASTContext.h`, `clang/ASTMatchers/ASTMatchFinder.h`.
- CN: 直接包含依赖: `DanglingHandleCheck.h`、`../utils/OptionsUtils.h`、`clang/AST/ASTContext.h`、`clang/ASTMatchers/ASTMatchFinder.h`。
- EN: Namespace context: `clang::tidy::bugprone`.
- CN: 命名空间上下文: `clang::tidy::bugprone`。
