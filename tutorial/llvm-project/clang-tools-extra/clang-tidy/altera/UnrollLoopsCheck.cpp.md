# UnrollLoopsCheck.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `clang-tools-extra/clang-tidy/altera/UnrollLoopsCheck.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `UnrollLoopsCheck` by registering AST matchers and producing diagnostics/fix-its.
- 用途 (CN): 通过注册 AST 匹配器并生成诊断/修复建议来实现 `UnrollLoopsCheck`。

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
   9 | #include "UnrollLoopsCheck.h"
  10 | #include "clang/AST/APValue.h"
  11 | #include "clang/AST/ASTContext.h"
  12 | #include "clang/AST/ASTTypeTraits.h"
  13 | #include "clang/AST/OperationKinds.h"
  14 | #include "clang/AST/ParentMapContext.h"
  15 | #include "clang/ASTMatchers/ASTMatchFinder.h"
  16 | #include <cmath>
  17 | 
```
- EN: The section imports dependencies such as `UnrollLoopsCheck.h`, `clang/AST/APValue.h`, `clang/AST/ASTContext.h`, `clang/AST/ASTTypeTraits.h` needed by this file.
- CN: 本段引入了 `UnrollLoopsCheck.h`、`clang/AST/APValue.h`、`clang/AST/ASTContext.h`、`clang/AST/ASTTypeTraits.h` 等依赖，供当前文件使用。

### Lines 18-21
```cpp
  18 | using namespace clang::ast_matchers;
  19 | 
  20 | namespace clang::tidy::altera {
  21 | 
```
- EN: Using-directives shorten qualified names so matcher/transformer code reads more directly.
- CN: using 指令缩短了限定名，使匹配器/变换器代码更直接。
- EN: Namespace scopes such as `clang::tidy::altera` place the symbols in their intended subsystem.
- CN: 诸如 `clang::tidy::altera` 这样的命名空间将符号放入预期的子系统中。

### Lines 22-25
```cpp
  22 | UnrollLoopsCheck::UnrollLoopsCheck(StringRef Name, ClangTidyContext *Context)
  23 |     : ClangTidyCheck(Name, Context),
  24 |       MaxLoopIterations(Options.get("MaxLoopIterations", 100U)) {}
  25 | 
```
- EN: This block continues the implementation with declarations or statements centered on `UnrollLoopsCheck::UnrollLoopsCheck(StringRef Name, ClangTidy`.
- CN: 这一段继续实现，围绕 `UnrollLoopsCheck::UnrollLoopsCheck(StringRef Name, ClangTidy` 展开声明或语句。

### Lines 26-35
```cpp
  26 | void UnrollLoopsCheck::registerMatchers(MatchFinder *Finder) {
  27 |   const auto HasLoopBound = hasDescendant(
  28 |       varDecl(matchesName("__end*"),
  29 |               hasDescendant(integerLiteral().bind("cxx_loop_bound"))));
  30 |   const auto CXXForRangeLoop =
  31 |       cxxForRangeStmt(anyOf(HasLoopBound, unless(HasLoopBound)));
  32 |   const auto AnyLoop = anyOf(forStmt(), whileStmt(), doStmt(), CXXForRangeLoop);
  33 |   Finder->addMatcher(
  34 |       stmt(AnyLoop, unless(hasDescendant(stmt(AnyLoop)))).bind("loop"), this);
  35 | }
```
- EN: This block defines AST matcher registration, i.e. the structural patterns the check will look for.
- CN: 这一段定义 AST 匹配器注册，也就是该检查要查找的结构模式。
- EN: Method definitions such as `UnrollLoopsCheck::registerMatchers` provide the concrete behavior declared elsewhere.
- CN: 诸如 `UnrollLoopsCheck::registerMatchers` 的方法定义给出了前面声明的具体行为。

### Lines 36-45
```cpp
  36 | 
  37 | void UnrollLoopsCheck::check(const MatchFinder::MatchResult &Result) {
  38 |   const auto *Loop = Result.Nodes.getNodeAs<Stmt>("loop");
  39 |   const auto *CXXLoopBound =
  40 |       Result.Nodes.getNodeAs<IntegerLiteral>("cxx_loop_bound");
  41 |   const ASTContext *Context = Result.Context;
  42 |   switch (unrollType(Loop, Result.Context)) {
  43 |   case NotUnrolled:
  44 |     diag(Loop->getBeginLoc(),
  45 |          "kernel performance could be improved by unrolling this loop with a "
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。
- EN: Method definitions such as `UnrollLoopsCheck::check` provide the concrete behavior declared elsewhere.
- CN: 诸如 `UnrollLoopsCheck::check` 的方法定义给出了前面声明的具体行为。

### Lines 46-55
```cpp
  46 |          "'#pragma unroll' directive");
  47 |     break;
  48 |   case PartiallyUnrolled:
  49 |     // Loop already partially unrolled, do nothing.
  50 |     break;
  51 |   case FullyUnrolled:
  52 |     if (hasKnownBounds(Loop, CXXLoopBound, Context)) {
  53 |       if (hasLargeNumIterations(Loop, CXXLoopBound, Context)) {
  54 |         diag(Loop->getBeginLoc(),
  55 |              "loop likely has a large number of iterations and thus "
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

### Lines 56-65
```cpp
  56 |              "cannot be fully unrolled; to partially unroll this loop, use "
  57 |              "the '#pragma unroll <num>' directive");
  58 |         return;
  59 |       }
  60 |       return;
  61 |     }
  62 |     if (isa<WhileStmt, DoStmt>(Loop)) {
  63 |       diag(Loop->getBeginLoc(),
  64 |            "full unrolling requested, but loop bounds may not be known; to "
  65 |            "partially unroll this loop, use the '#pragma unroll <num>' "
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

### Lines 66-75
```cpp
  66 |            "directive",
  67 |            DiagnosticIDs::Note);
  68 |       break;
  69 |     }
  70 |     diag(Loop->getBeginLoc(),
  71 |          "full unrolling requested, but loop bounds are not known; to "
  72 |          "partially unroll this loop, use the '#pragma unroll <num>' "
  73 |          "directive");
  74 |     break;
  75 |   }
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

### Lines 76-85
```cpp
  76 | }
  77 | 
  78 | enum UnrollLoopsCheck::UnrollType
  79 | UnrollLoopsCheck::unrollType(const Stmt *Statement, ASTContext *Context) {
  80 |   const DynTypedNodeList Parents = Context->getParents<Stmt>(*Statement);
  81 |   for (const DynTypedNode &Parent : Parents) {
  82 |     const auto *ParentStmt = Parent.get<AttributedStmt>();
  83 |     if (!ParentStmt)
  84 |       continue;
  85 |     for (const Attr *Attribute : ParentStmt->getAttrs()) {
```
- EN: This block continues the implementation with declarations or statements centered on `}`.
- CN: 这一段继续实现，围绕 `}` 展开声明或语句。

### Lines 86-95
```cpp
  86 |       const auto *LoopHint = dyn_cast<LoopHintAttr>(Attribute);
  87 |       if (!LoopHint)
  88 |         continue;
  89 |       switch (LoopHint->getState()) {
  90 |       case LoopHintAttr::Numeric:
  91 |         return PartiallyUnrolled;
  92 |       case LoopHintAttr::Disable:
  93 |         return NotUnrolled;
  94 |       case LoopHintAttr::Full:
  95 |         return FullyUnrolled;
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 96-105
```cpp
  96 |       case LoopHintAttr::Enable:
  97 |         return FullyUnrolled;
  98 |       case LoopHintAttr::AssumeSafety:
  99 |         return NotUnrolled;
 100 |       case LoopHintAttr::FixedWidth:
 101 |         return NotUnrolled;
 102 |       case LoopHintAttr::ScalableWidth:
 103 |         return NotUnrolled;
 104 |       }
 105 |     }
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 106-109
```cpp
 106 |   }
 107 |   return NotUnrolled;
 108 | }
 109 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 110-119
```cpp
 110 | bool UnrollLoopsCheck::hasKnownBounds(const Stmt *Statement,
 111 |                                       const IntegerLiteral *CXXLoopBound,
 112 |                                       const ASTContext *Context) {
 113 |   if (isa<CXXForRangeStmt>(Statement))
 114 |     return CXXLoopBound != nullptr;
 115 |   // Too many possibilities in a while statement, so always recommend partial
 116 |   // unrolling for these.
 117 |   if (isa<WhileStmt, DoStmt>(Statement))
 118 |     return false;
 119 |   // The last loop type is a for loop.
```
- EN: Method definitions such as `UnrollLoopsCheck::hasKnownBounds` provide the concrete behavior declared elsewhere.
- CN: 诸如 `UnrollLoopsCheck::hasKnownBounds` 的方法定义给出了前面声明的具体行为。
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 120-129
```cpp
 120 |   const auto *ForLoop = cast<ForStmt>(Statement);
 121 |   const Stmt *Initializer = ForLoop->getInit();
 122 |   const Expr *Conditional = ForLoop->getCond();
 123 |   const Expr *Increment = ForLoop->getInc();
 124 |   if (!Initializer || !Conditional || !Increment)
 125 |     return false;
 126 |   // If the loop variable value isn't known, loop bounds are unknown.
 127 |   if (const auto *InitDeclStatement = dyn_cast<DeclStmt>(Initializer)) {
 128 |     if (const auto *VariableDecl =
 129 |             dyn_cast<VarDecl>(InitDeclStatement->getSingleDecl())) {
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 130-139
```cpp
 130 |       const APValue *Evaluation = VariableDecl->evaluateValue();
 131 |       if (!Evaluation || !Evaluation->hasValue())
 132 |         return false;
 133 |     }
 134 |   }
 135 |   // If increment is unary and not one of ++ and --, loop bounds are unknown.
 136 |   if (const auto *Op = dyn_cast<UnaryOperator>(Increment))
 137 |     if (!Op->isIncrementDecrementOp())
 138 |       return false;
 139 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 140-148
```cpp
 140 |   if (const auto *BinaryOp = dyn_cast<BinaryOperator>(Conditional)) {
 141 |     const Expr *LHS = BinaryOp->getLHS();
 142 |     const Expr *RHS = BinaryOp->getRHS();
 143 |     // If both sides are value dependent or constant, loop bounds are unknown.
 144 |     return LHS->isEvaluatable(*Context) != RHS->isEvaluatable(*Context);
 145 |   }
 146 |   return false; // If it's not a binary operator, loop bounds are unknown.
 147 | }
 148 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 149-158
```cpp
 149 | const Expr *UnrollLoopsCheck::getCondExpr(const Stmt *Statement) {
 150 |   if (const auto *ForLoop = dyn_cast<ForStmt>(Statement))
 151 |     return ForLoop->getCond();
 152 |   if (const auto *WhileLoop = dyn_cast<WhileStmt>(Statement))
 153 |     return WhileLoop->getCond();
 154 |   if (const auto *DoWhileLoop = dyn_cast<DoStmt>(Statement))
 155 |     return DoWhileLoop->getCond();
 156 |   if (const auto *CXXRangeLoop = dyn_cast<CXXForRangeStmt>(Statement))
 157 |     return CXXRangeLoop->getCond();
 158 |   llvm_unreachable("Unknown loop");
```
- EN: Method definitions such as `UnrollLoopsCheck::getCondExpr` provide the concrete behavior declared elsewhere.
- CN: 诸如 `UnrollLoopsCheck::getCondExpr` 的方法定义给出了前面声明的具体行为。
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 159-168
```cpp
 159 | }
 160 | 
 161 | bool UnrollLoopsCheck::hasLargeNumIterations(const Stmt *Statement,
 162 |                                              const IntegerLiteral *CXXLoopBound,
 163 |                                              const ASTContext *Context) {
 164 |   // Because hasKnownBounds is called before this, if this is true, then
 165 |   // CXXLoopBound is also matched.
 166 |   if (isa<CXXForRangeStmt>(Statement)) {
 167 |     assert(CXXLoopBound && "CXX ranged for loop has no loop bound");
 168 |     return exprHasLargeNumIterations(CXXLoopBound, Context);
```
- EN: Method definitions such as `UnrollLoopsCheck::hasLargeNumIterations` provide the concrete behavior declared elsewhere.
- CN: 诸如 `UnrollLoopsCheck::hasLargeNumIterations` 的方法定义给出了前面声明的具体行为。
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 169-178
```cpp
 169 |   }
 170 |   const auto *ForLoop = cast<ForStmt>(Statement);
 171 |   const Stmt *Initializer = ForLoop->getInit();
 172 |   const Expr *Conditional = ForLoop->getCond();
 173 |   const Expr *Increment = ForLoop->getInc();
 174 |   int InitValue = 0;
 175 |   // If the loop variable value isn't known, we can't know the loop bounds.
 176 |   if (const auto *InitDeclStatement = dyn_cast<DeclStmt>(Initializer)) {
 177 |     if (const auto *VariableDecl =
 178 |             dyn_cast<VarDecl>(InitDeclStatement->getSingleDecl())) {
```
- EN: This block continues the implementation with declarations or statements centered on `}`.
- CN: 这一段继续实现，围绕 `}` 展开声明或语句。

### Lines 179-185
```cpp
 179 |       APValue *Evaluation = VariableDecl->evaluateValue();
 180 |       if (!Evaluation || !Evaluation->isInt())
 181 |         return true;
 182 |       InitValue = Evaluation->getInt().getExtValue();
 183 |     }
 184 |   }
 185 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 186-190
```cpp
 186 |   int EndValue = 0;
 187 |   const auto *BinaryOp = cast<BinaryOperator>(Conditional);
 188 |   if (!extractValue(EndValue, BinaryOp, Context))
 189 |     return true;
 190 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 191-200
```cpp
 191 |   double Iterations = 0.0;
 192 | 
 193 |   // If increment is unary and not one of ++, --, we can't know the loop bounds.
 194 |   if (const auto *Op = dyn_cast<UnaryOperator>(Increment)) {
 195 |     if (Op->isIncrementOp())
 196 |       Iterations = EndValue - InitValue;
 197 |     else if (Op->isDecrementOp())
 198 |       Iterations = InitValue - EndValue;
 199 |     else
 200 |       llvm_unreachable("Unary operator neither increment nor decrement");
```
- EN: This block continues the implementation with declarations or statements centered on `double Iterations = 0.0;`.
- CN: 这一段继续实现，围绕 `double Iterations = 0.0;` 展开声明或语句。

### Lines 201-210
```cpp
 201 |   }
 202 | 
 203 |   // If increment is binary and not one of +, -, *, /, we can't know the loop
 204 |   // bounds.
 205 |   if (const auto *Op = dyn_cast<BinaryOperator>(Increment)) {
 206 |     int ConstantValue = 0;
 207 |     if (!extractValue(ConstantValue, Op, Context))
 208 |       return true;
 209 |     switch (Op->getOpcode()) {
 210 |     case BO_AddAssign:
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 211-220
```cpp
 211 |       Iterations =
 212 |           std::ceil(static_cast<float>(EndValue - InitValue) / ConstantValue);
 213 |       break;
 214 |     case BO_SubAssign:
 215 |       Iterations =
 216 |           std::ceil(static_cast<float>(InitValue - EndValue) / ConstantValue);
 217 |       break;
 218 |     case BO_MulAssign:
 219 |       Iterations = 1 + ((std::log(static_cast<double>(EndValue)) -
 220 |                          std::log(static_cast<double>(InitValue))) /
```
- EN: Method definitions such as `std::ceil`, `std::log` provide the concrete behavior declared elsewhere.
- CN: 诸如 `std::ceil`、`std::log` 的方法定义给出了前面声明的具体行为。

### Lines 221-230
```cpp
 221 |                         std::log(static_cast<double>(ConstantValue)));
 222 |       break;
 223 |     case BO_DivAssign:
 224 |       Iterations = 1 + ((std::log(static_cast<double>(InitValue)) -
 225 |                          std::log(static_cast<double>(EndValue))) /
 226 |                         std::log(static_cast<double>(ConstantValue)));
 227 |       break;
 228 |     default:
 229 |       // All other operators are not handled; assume large bounds.
 230 |       return true;
```
- EN: Method definitions such as `std::log` provide the concrete behavior declared elsewhere.
- CN: 诸如 `std::log` 的方法定义给出了前面声明的具体行为。
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 231-235
```cpp
 231 |     }
 232 |   }
 233 |   return Iterations > MaxLoopIterations;
 234 | }
 235 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 236-245
```cpp
 236 | bool UnrollLoopsCheck::extractValue(int &Value, const BinaryOperator *Op,
 237 |                                     const ASTContext *Context) {
 238 |   const Expr *LHS = Op->getLHS();
 239 |   const Expr *RHS = Op->getRHS();
 240 |   Expr::EvalResult Result;
 241 |   if (LHS->isEvaluatable(*Context))
 242 |     LHS->EvaluateAsRValue(Result, *Context);
 243 |   else if (RHS->isEvaluatable(*Context))
 244 |     RHS->EvaluateAsRValue(Result, *Context);
 245 |   else
```
- EN: Method definitions such as `UnrollLoopsCheck::extractValue` provide the concrete behavior declared elsewhere.
- CN: 诸如 `UnrollLoopsCheck::extractValue` 的方法定义给出了前面声明的具体行为。

### Lines 246-253
```cpp
 246 |     return false; // Cannot evaluate either side.
 247 |   if (!Result.Val.isInt())
 248 |     return false; // Cannot check number of iterations, return false to be
 249 |                   // safe.
 250 |   Value = Result.Val.getInt().getExtValue();
 251 |   return true;
 252 | }
 253 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 254-263
```cpp
 254 | bool UnrollLoopsCheck::exprHasLargeNumIterations(
 255 |     const Expr *Expression, const ASTContext *Context) const {
 256 |   Expr::EvalResult Result;
 257 |   if (Expression->EvaluateAsRValue(Result, *Context)) {
 258 |     if (!Result.Val.isInt())
 259 |       return false; // Cannot check number of iterations, return false to be
 260 |                     // safe.
 261 |     // The following assumes values go from 0 to Val in increments of 1.
 262 |     return Result.Val.getInt() > MaxLoopIterations;
 263 |   }
```
- EN: Method definitions such as `UnrollLoopsCheck::exprHasLargeNumIterations` provide the concrete behavior declared elsewhere.
- CN: 诸如 `UnrollLoopsCheck::exprHasLargeNumIterations` 的方法定义给出了前面声明的具体行为。
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 264-268
```cpp
 264 |   // Cannot evaluate Expression as an r-value, so cannot check number of
 265 |   // iterations.
 266 |   return false;
 267 | }
 268 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 269-273
```cpp
 269 | void UnrollLoopsCheck::storeOptions(ClangTidyOptions::OptionMap &Opts) {
 270 |   Options.store(Opts, "MaxLoopIterations", MaxLoopIterations);
 271 | }
 272 | 
 273 | } // namespace clang::tidy::altera
```
- EN: Method definitions such as `UnrollLoopsCheck::storeOptions` provide the concrete behavior declared elsewhere.
- CN: 诸如 `UnrollLoopsCheck::storeOptions` 的方法定义给出了前面声明的具体行为。

## Key Concepts / 关键概念
- clang-tidy check lifecycle / clang-tidy 检查生命周期
- AST matcher DSL / AST 匹配器 DSL
- diagnostics and fix-its / 诊断与修复建议
- namespace scoping / 命名空间作用域

## Dependencies / 依赖关系
- EN: Direct includes: `UnrollLoopsCheck.h`, `clang/AST/APValue.h`, `clang/AST/ASTContext.h`, `clang/AST/ASTTypeTraits.h`, `clang/AST/OperationKinds.h`, `clang/AST/ParentMapContext.h`, `clang/ASTMatchers/ASTMatchFinder.h`, `cmath`.
- CN: 直接包含依赖: `UnrollLoopsCheck.h`、`clang/AST/APValue.h`、`clang/AST/ASTContext.h`、`clang/AST/ASTTypeTraits.h`、`clang/AST/OperationKinds.h`、`clang/AST/ParentMapContext.h`、`clang/ASTMatchers/ASTMatchFinder.h`、`cmath`。
- EN: Namespace context: `clang::tidy::altera`.
- CN: 命名空间上下文: `clang::tidy::altera`。
