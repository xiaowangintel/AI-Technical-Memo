# TimeSubtractionCheck.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `clang-tools-extra/clang-tidy/abseil/TimeSubtractionCheck.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `TimeSubtractionCheck` by registering AST matchers and producing diagnostics/fix-its.
- 用途 (CN): 通过注册 AST 匹配器并生成诊断/修复建议来实现 `TimeSubtractionCheck`。

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
   9 | #include "TimeSubtractionCheck.h"
  10 | #include "DurationRewriter.h"
  11 | #include "clang/AST/ASTContext.h"
  12 | #include "clang/ASTMatchers/ASTMatchFinder.h"
  13 | #include "clang/Lex/Lexer.h"
  14 | #include <optional>
  15 | 
```
- EN: The section imports dependencies such as `TimeSubtractionCheck.h`, `DurationRewriter.h`, `clang/AST/ASTContext.h`, `clang/ASTMatchers/ASTMatchFinder.h` needed by this file.
- CN: 本段引入了 `TimeSubtractionCheck.h`、`DurationRewriter.h`、`clang/AST/ASTContext.h`、`clang/ASTMatchers/ASTMatchFinder.h` 等依赖，供当前文件使用。

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

### Lines 20-28
```cpp
  20 | // Returns `true` if `Range` is inside a macro definition.
  21 | static bool insideMacroDefinition(const MatchFinder::MatchResult &Result,
  22 |                                   SourceRange Range) {
  23 |   return !Lexer::makeFileCharRange(CharSourceRange::getCharRange(Range),
  24 |                                    *Result.SourceManager,
  25 |                                    Result.Context->getLangOpts())
  26 |               .isValid();
  27 | }
  28 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 29-38
```cpp
  29 | static bool isConstructorAssignment(const MatchFinder::MatchResult &Result,
  30 |                                     const Expr *Node) {
  31 |   // For C++14 and earlier there are elidable constructors that must be matched
  32 |   // in hasParent. The elidable constructors do not exist in C++17 and later and
  33 |   // therefore an additional check that does not match against the elidable
  34 |   // constructors are needed for this case.
  35 |   return selectFirst<const Expr>(
  36 |              "e",
  37 |              match(expr(anyOf(
  38 |                        callExpr(hasParent(materializeTemporaryExpr(hasParent(
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 39-45
```cpp
  39 |                                     cxxConstructExpr(hasParent(exprWithCleanups(
  40 |                                         hasParent(varDecl()))))))))
  41 |                            .bind("e"),
  42 |                        callExpr(hasParent(varDecl())).bind("e"))),
  43 |                    *Node, *Result.Context)) != nullptr;
  44 | }
  45 | 
```
- EN: This block continues the implementation with declarations or statements centered on `cxxConstructExpr(hasParent(exprWithCleanups(`.
- CN: 这一段继续实现，围绕 `cxxConstructExpr(hasParent(exprWithCleanups(` 展开声明或语句。

### Lines 46-55
```cpp
  46 | static bool isArgument(const MatchFinder::MatchResult &Result,
  47 |                        const Expr *Node) {
  48 |   // For the same reason as in isConstructorAssignment two AST shapes need to be
  49 |   // matched here.
  50 |   return selectFirst<const Expr>(
  51 |              "e",
  52 |              match(
  53 |                  expr(anyOf(
  54 |                      expr(hasParent(materializeTemporaryExpr(
  55 |                               hasParent(cxxConstructExpr(
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 56-64
```cpp
  56 |                                   hasParent(callExpr()),
  57 |                                   unless(hasParent(cxxOperatorCallExpr())))))))
  58 |                          .bind("e"),
  59 |                      expr(hasParent(callExpr()),
  60 |                           unless(hasParent(cxxOperatorCallExpr())))
  61 |                          .bind("e"))),
  62 |                  *Node, *Result.Context)) != nullptr;
  63 | }
  64 | 
```
- EN: This block continues the implementation with declarations or statements centered on `hasParent(callExpr()),`.
- CN: 这一段继续实现，围绕 `hasParent(callExpr()),` 展开声明或语句。

### Lines 65-74
```cpp
  65 | static bool isReturn(const MatchFinder::MatchResult &Result, const Expr *Node) {
  66 |   // For the same reason as in isConstructorAssignment two AST shapes need to be
  67 |   // matched here.
  68 |   return selectFirst<const Expr>(
  69 |              "e",
  70 |              match(expr(anyOf(
  71 |                        expr(hasParent(materializeTemporaryExpr(hasParent(
  72 |                                 cxxConstructExpr(hasParent(exprWithCleanups(
  73 |                                     hasParent(returnStmt()))))))))
  74 |                            .bind("e"),
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 75-78
```cpp
  75 |                        expr(hasParent(returnStmt())).bind("e"))),
  76 |                    *Node, *Result.Context)) != nullptr;
  77 | }
  78 | 
```
- EN: This block continues the implementation with declarations or statements centered on `expr(hasParent(returnStmt())).bind("e"))),`.
- CN: 这一段继续实现，围绕 `expr(hasParent(returnStmt())).bind("e"))),` 展开声明或语句。

### Lines 79-86
```cpp
  79 | static bool parensRequired(const MatchFinder::MatchResult &Result,
  80 |                            const Expr *Node) {
  81 |   // TODO: Figure out any more contexts in which we can omit the surrounding
  82 |   // parentheses.
  83 |   return !(isConstructorAssignment(Result, Node) || isArgument(Result, Node) ||
  84 |            isReturn(Result, Node));
  85 | }
  86 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 87-92
```cpp
  87 | void TimeSubtractionCheck::emitDiagnostic(const Expr *Node,
  88 |                                           StringRef Replacement) {
  89 |   diag(Node->getBeginLoc(), "perform subtraction in the time domain")
  90 |       << FixItHint::CreateReplacement(Node->getSourceRange(), Replacement);
  91 | }
  92 | 
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。
- EN: Method definitions such as `TimeSubtractionCheck::emitDiagnostic`, `FixItHint::CreateReplacement` provide the concrete behavior declared elsewhere.
- CN: 诸如 `TimeSubtractionCheck::emitDiagnostic`、`FixItHint::CreateReplacement` 的方法定义给出了前面声明的具体行为。

### Lines 93-99
```cpp
  93 | void TimeSubtractionCheck::registerMatchers(MatchFinder *Finder) {
  94 |   for (const char *ScaleName :
  95 |        {"Hours", "Minutes", "Seconds", "Millis", "Micros", "Nanos"}) {
  96 |     const std::string TimeInverse = (llvm::Twine("ToUnix") + ScaleName).str();
  97 |     std::optional<DurationScale> Scale = getScaleForTimeInverse(TimeInverse);
  98 |     assert(Scale && "Unknown scale encountered");
  99 | 
```
- EN: This block defines AST matcher registration, i.e. the structural patterns the check will look for.
- CN: 这一段定义 AST 匹配器注册，也就是该检查要查找的结构模式。
- EN: Method definitions such as `TimeSubtractionCheck::registerMatchers` provide the concrete behavior declared elsewhere.
- CN: 诸如 `TimeSubtractionCheck::registerMatchers` 的方法定义给出了前面声明的具体行为。

### Lines 100-103
```cpp
 100 |     auto TimeInverseMatcher = callExpr(callee(
 101 |         functionDecl(hasName((llvm::Twine("::absl::") + TimeInverse).str()))
 102 |             .bind("func_decl")));
 103 | 
```
- EN: This block continues the implementation with declarations or statements centered on `auto TimeInverseMatcher = callExpr(callee(`.
- CN: 这一段继续实现，围绕 `auto TimeInverseMatcher = callExpr(callee(` 展开声明或语句。

### Lines 104-113
```cpp
 104 |     // Match the cases where we know that the result is a 'Duration' and the
 105 |     // first argument is a 'Time'. Just knowing the type of the first operand
 106 |     // is not sufficient, since the second operand could be either a 'Time' or
 107 |     // a 'Duration'. If we know the result is a 'Duration', we can then infer
 108 |     // that the second operand must be a 'Time'.
 109 |     auto CallMatcher =
 110 |         callExpr(
 111 |             callee(functionDecl(hasName(getDurationFactoryForScale(*Scale)))),
 112 |             hasArgument(0, binaryOperator(hasOperatorName("-"),
 113 |                                           hasLHS(TimeInverseMatcher))
```
- EN: This block continues the implementation with declarations or statements centered on `// Match the cases where we know that the result is a 'Durat`.
- CN: 这一段继续实现，围绕 `// Match the cases where we know that the result is a 'Durat` 展开声明或语句。

### Lines 114-117
```cpp
 114 |                                .bind("binop")))
 115 |             .bind("outer_call");
 116 |     Finder->addMatcher(CallMatcher, this);
 117 | 
```
- EN: This block continues the implementation with declarations or statements centered on `.bind("binop")))`.
- CN: 这一段继续实现，围绕 `.bind("binop")))` 展开声明或语句。

### Lines 118-127
```cpp
 118 |     // Match cases where we know the second operand is a 'Time'. Since
 119 |     // subtracting a 'Time' from a 'Duration' is not defined, in these cases,
 120 |     // we always know the first operand is a 'Time' if the second is a 'Time'.
 121 |     auto OperandMatcher =
 122 |         binaryOperator(hasOperatorName("-"), hasRHS(TimeInverseMatcher))
 123 |             .bind("binop");
 124 |     Finder->addMatcher(OperandMatcher, this);
 125 |   }
 126 | }
 127 | 
```
- EN: This block continues the implementation with declarations or statements centered on `// Match cases where we know the second operand is a 'Time'.`.
- CN: 这一段继续实现，围绕 `// Match cases where we know the second operand is a 'Time'.` 展开声明或语句。

### Lines 128-134
```cpp
 128 | void TimeSubtractionCheck::check(const MatchFinder::MatchResult &Result) {
 129 |   const auto *BinOp = Result.Nodes.getNodeAs<BinaryOperator>("binop");
 130 |   const std::string InverseName =
 131 |       Result.Nodes.getNodeAs<FunctionDecl>("func_decl")->getNameAsString();
 132 |   if (insideMacroDefinition(Result, BinOp->getSourceRange()))
 133 |     return;
 134 | 
```
- EN: Method definitions such as `TimeSubtractionCheck::check` provide the concrete behavior declared elsewhere.
- CN: 诸如 `TimeSubtractionCheck::check` 的方法定义给出了前面声明的具体行为。

### Lines 135-138
```cpp
 135 |   std::optional<DurationScale> Scale = getScaleForTimeInverse(InverseName);
 136 |   if (!Scale)
 137 |     return;
 138 | 
```
- EN: This block continues the implementation with declarations or statements centered on `std::optional<DurationScale> Scale = getScaleForTimeInverse(`.
- CN: 这一段继续实现，围绕 `std::optional<DurationScale> Scale = getScaleForTimeInverse(` 展开声明或语句。

### Lines 139-143
```cpp
 139 |   const auto *OuterCall = Result.Nodes.getNodeAs<CallExpr>("outer_call");
 140 |   if (OuterCall) {
 141 |     if (insideMacroDefinition(Result, OuterCall->getSourceRange()))
 142 |       return;
 143 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const auto *OuterCall = Result.Nodes.getNodeAs<CallExpr>("ou`.
- CN: 这一段继续实现，围绕 `const auto *OuterCall = Result.Nodes.getNodeAs<CallExpr>("ou` 展开声明或语句。

### Lines 144-148
```cpp
 144 |     // We're working with the first case of matcher, and need to replace the
 145 |     // entire 'Duration' factory call. (Which also means being careful about
 146 |     // our order-of-operations and optionally putting in some parenthesis.
 147 |     const bool NeedParens = parensRequired(Result, OuterCall);
 148 | 
```
- EN: This block continues the implementation with declarations or statements centered on `// We're working with the first case of matcher, and need to`.
- CN: 这一段继续实现，围绕 `// We're working with the first case of matcher, and need to` 展开声明或语句。

### Lines 149-158
```cpp
 149 |     emitDiagnostic(
 150 |         OuterCall,
 151 |         (llvm::Twine(NeedParens ? "(" : "") +
 152 |          rewriteExprFromNumberToTime(Result, *Scale, BinOp->getLHS()) + " - " +
 153 |          rewriteExprFromNumberToTime(Result, *Scale, BinOp->getRHS()) +
 154 |          (NeedParens ? ")" : ""))
 155 |             .str());
 156 |   } else {
 157 |     // We're working with the second case of matcher, and either just need to
 158 |     // change the arguments, or perhaps remove an outer function call. In the
```
- EN: This block continues the implementation with declarations or statements centered on `emitDiagnostic(`.
- CN: 这一段继续实现，围绕 `emitDiagnostic(` 展开声明或语句。

### Lines 159-168
```cpp
 159 |     // latter case (addressed first), we also need to worry about parenthesis.
 160 |     const auto *MaybeCallArg = selectFirst<const CallExpr>(
 161 |         "arg", match(expr(hasAncestor(
 162 |                          callExpr(callee(functionDecl(hasName(
 163 |                                       getDurationFactoryForScale(*Scale)))))
 164 |                              .bind("arg"))),
 165 |                      *BinOp, *Result.Context));
 166 |     if (MaybeCallArg && MaybeCallArg->getArg(0)->IgnoreImpCasts() == BinOp &&
 167 |         !insideMacroDefinition(Result, MaybeCallArg->getSourceRange())) {
 168 |       // Handle the case where the matched expression is inside a call which
```
- EN: This block continues the implementation with declarations or statements centered on `// latter case (addressed first), we also need to worry abou`.
- CN: 这一段继续实现，围绕 `// latter case (addressed first), we also need to worry abou` 展开声明或语句。

### Lines 169-173
```cpp
 169 |       // converts it from the inverse to a Duration.  In this case, we replace
 170 |       // the outer with just the subtraction expression, which gives the right
 171 |       // type and scale, taking care again about parenthesis.
 172 |       const bool NeedParens = parensRequired(Result, MaybeCallArg);
 173 | 
```
- EN: This block continues the implementation with declarations or statements centered on `// converts it from the inverse to a Duration.  In this case`.
- CN: 这一段继续实现，围绕 `// converts it from the inverse to a Duration.  In this case` 展开声明或语句。

### Lines 174-183
```cpp
 174 |       emitDiagnostic(
 175 |           MaybeCallArg,
 176 |           (llvm::Twine(NeedParens ? "(" : "") +
 177 |            rewriteExprFromNumberToTime(Result, *Scale, BinOp->getLHS()) +
 178 |            " - " +
 179 |            rewriteExprFromNumberToTime(Result, *Scale, BinOp->getRHS()) +
 180 |            (NeedParens ? ")" : ""))
 181 |               .str());
 182 |     } else {
 183 |       // In the last case, just convert the arguments and wrap the result in
```
- EN: This block continues the implementation with declarations or statements centered on `emitDiagnostic(`.
- CN: 这一段继续实现，围绕 `emitDiagnostic(` 展开声明或语句。

### Lines 184-193
```cpp
 184 |       // the correct inverse function.
 185 |       emitDiagnostic(
 186 |           BinOp,
 187 |           (llvm::Twine(
 188 |                getDurationInverseForScale(*Scale).second.str().substr(2)) +
 189 |            "(" + rewriteExprFromNumberToTime(Result, *Scale, BinOp->getLHS()) +
 190 |            " - " +
 191 |            rewriteExprFromNumberToTime(Result, *Scale, BinOp->getRHS()) + ")")
 192 |               .str());
 193 |     }
```
- EN: This block continues the implementation with declarations or statements centered on `// the correct inverse function.`.
- CN: 这一段继续实现，围绕 `// the correct inverse function.` 展开声明或语句。

### Lines 194-197
```cpp
 194 |   }
 195 | }
 196 | 
 197 | } // namespace clang::tidy::abseil
```
- EN: This block continues the implementation with declarations or statements centered on `}`.
- CN: 这一段继续实现，围绕 `}` 展开声明或语句。

## Key Concepts / 关键概念
- AST matcher DSL / AST 匹配器 DSL
- diagnostics and fix-its / 诊断与修复建议
- namespace scoping / 命名空间作用域

## Dependencies / 依赖关系
- EN: Direct includes: `TimeSubtractionCheck.h`, `DurationRewriter.h`, `clang/AST/ASTContext.h`, `clang/ASTMatchers/ASTMatchFinder.h`, `clang/Lex/Lexer.h`, `optional`.
- CN: 直接包含依赖: `TimeSubtractionCheck.h`、`DurationRewriter.h`、`clang/AST/ASTContext.h`、`clang/ASTMatchers/ASTMatchFinder.h`、`clang/Lex/Lexer.h`、`optional`。
- EN: Namespace context: `clang::tidy::abseil`.
- CN: 命名空间上下文: `clang::tidy::abseil`。
