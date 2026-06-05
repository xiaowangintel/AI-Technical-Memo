# SwappedArgumentsCheck.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `clang-tools-extra/clang-tidy/bugprone/SwappedArgumentsCheck.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `SwappedArgumentsCheck` by registering AST matchers and producing diagnostics/fix-its.
- 用途 (CN): 通过注册 AST 匹配器并生成诊断/修复建议来实现 `SwappedArgumentsCheck`。

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
   9 | #include "SwappedArgumentsCheck.h"
  10 | #include "clang/AST/ASTContext.h"
  11 | #include "clang/Lex/Lexer.h"
  12 | #include "clang/Tooling/FixIt.h"
  13 | #include "llvm/ADT/SmallPtrSet.h"
  14 | 
```
- EN: The section imports dependencies such as `SwappedArgumentsCheck.h`, `clang/AST/ASTContext.h`, `clang/Lex/Lexer.h`, `clang/Tooling/FixIt.h` needed by this file.
- CN: 本段引入了 `SwappedArgumentsCheck.h`、`clang/AST/ASTContext.h`、`clang/Lex/Lexer.h`、`clang/Tooling/FixIt.h` 等依赖，供当前文件使用。

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

### Lines 19-23
```cpp
  19 | void SwappedArgumentsCheck::registerMatchers(MatchFinder *Finder) {
  20 |   Finder->addMatcher(callExpr(unless(isInTemplateInstantiation())).bind("call"),
  21 |                      this);
  22 | }
  23 | 
```
- EN: This block defines AST matcher registration, i.e. the structural patterns the check will look for.
- CN: 这一段定义 AST 匹配器注册，也就是该检查要查找的结构模式。
- EN: Method definitions such as `SwappedArgumentsCheck::registerMatchers` provide the concrete behavior declared elsewhere.
- CN: 诸如 `SwappedArgumentsCheck::registerMatchers` 的方法定义给出了前面声明的具体行为。

### Lines 24-33
```cpp
  24 | /// Look through lvalue to rvalue and nop casts. This filters out
  25 | /// implicit conversions that have no effect on the input but block our view for
  26 | /// other implicit casts.
  27 | static const Expr *ignoreNoOpCasts(const Expr *E) {
  28 |   if (auto *Cast = dyn_cast<CastExpr>(E))
  29 |     if (Cast->getCastKind() == CK_LValueToRValue ||
  30 |         Cast->getCastKind() == CK_NoOp)
  31 |       return ignoreNoOpCasts(Cast->getSubExpr());
  32 |   return E;
  33 | }
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 34-43
```cpp
  34 | 
  35 | /// Restrict the warning to implicit casts that are most likely
  36 | /// accidental. User defined or integral conversions fit in this category,
  37 | /// lvalue to rvalue or derived to base does not.
  38 | static bool isImplicitCastCandidate(const CastExpr *Cast) {
  39 |   return Cast->getCastKind() == CK_UserDefinedConversion ||
  40 |          Cast->getCastKind() == CK_FloatingToBoolean ||
  41 |          Cast->getCastKind() == CK_FloatingToIntegral ||
  42 |          Cast->getCastKind() == CK_IntegralToBoolean ||
  43 |          Cast->getCastKind() == CK_IntegralToFloating ||
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 44-49
```cpp
  44 |          Cast->getCastKind() == CK_MemberPointerToBoolean ||
  45 |          Cast->getCastKind() == CK_PointerToBoolean ||
  46 |          (Cast->getCastKind() == CK_IntegralCast &&
  47 |           Cast->getSubExpr()->getType()->isBooleanType());
  48 | }
  49 | 
```
- EN: This block continues the implementation with declarations or statements centered on `Cast->getCastKind() == CK_MemberPointerToBoolean ||`.
- CN: 这一段继续实现，围绕 `Cast->getCastKind() == CK_MemberPointerToBoolean ||` 展开声明或语句。

### Lines 50-53
```cpp
  50 | static bool areTypesSemiEqual(const QualType L, const QualType R) {
  51 |   if (L == R)
  52 |     return true;
  53 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 54-61
```cpp
  54 |   if (!L->isBuiltinType() || !R->isBuiltinType())
  55 |     return false;
  56 | 
  57 |   return (L->isFloatingType() && R->isFloatingType()) ||
  58 |          (L->isIntegerType() && R->isIntegerType()) ||
  59 |          (L->isBooleanType() && R->isBooleanType());
  60 | }
  61 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 62-68
```cpp
  62 | static bool areArgumentsPotentiallySwapped(const QualType LTo,
  63 |                                            const QualType RTo,
  64 |                                            const QualType LFrom,
  65 |                                            const QualType RFrom) {
  66 |   if (LTo == RTo || LFrom == RFrom)
  67 |     return false;
  68 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 69-72
```cpp
  69 |   const bool REq = areTypesSemiEqual(RTo, LFrom);
  70 |   if (LTo == RFrom && REq)
  71 |     return true;
  72 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 73-76
```cpp
  73 |   const bool LEq = areTypesSemiEqual(LTo, RFrom);
  74 |   if (RTo == LFrom && LEq)
  75 |     return true;
  76 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 77-82
```cpp
  77 |   if (REq && LEq && !areTypesSemiEqual(RTo, LTo))
  78 |     return true;
  79 | 
  80 |   return false;
  81 | }
  82 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 83-86
```cpp
  83 | void SwappedArgumentsCheck::check(const MatchFinder::MatchResult &Result) {
  84 |   const ASTContext &Ctx = *Result.Context;
  85 |   const auto *Call = Result.Nodes.getNodeAs<CallExpr>("call");
  86 | 
```
- EN: Method definitions such as `SwappedArgumentsCheck::check` provide the concrete behavior declared elsewhere.
- CN: 诸如 `SwappedArgumentsCheck::check` 的方法定义给出了前面声明的具体行为。

### Lines 87-91
```cpp
  87 |   llvm::SmallPtrSet<const Expr *, 4> UsedArgs;
  88 |   for (unsigned I = 1, E = Call->getNumArgs(); I < E; ++I) {
  89 |     const Expr *LHS = Call->getArg(I - 1);
  90 |     const Expr *RHS = Call->getArg(I);
  91 | 
```
- EN: This block continues the implementation with declarations or statements centered on `llvm::SmallPtrSet<const Expr *, 4> UsedArgs;`.
- CN: 这一段继续实现，围绕 `llvm::SmallPtrSet<const Expr *, 4> UsedArgs;` 展开声明或语句。

### Lines 92-96
```cpp
  92 |     // Only need to check RHS, as LHS has already been covered. We don't want to
  93 |     // emit two warnings for a single argument.
  94 |     if (UsedArgs.contains(RHS))
  95 |       continue;
  96 | 
```
- EN: This block continues the implementation with declarations or statements centered on `// Only need to check RHS, as LHS has already been covered. `.
- CN: 这一段继续实现，围绕 `// Only need to check RHS, as LHS has already been covered. ` 展开声明或语句。

### Lines 97-105
```cpp
  97 |     const auto *LHSCast = dyn_cast<ImplicitCastExpr>(ignoreNoOpCasts(LHS));
  98 |     const auto *RHSCast = dyn_cast<ImplicitCastExpr>(ignoreNoOpCasts(RHS));
  99 | 
 100 |     // Look if this is a potentially swapped argument pair. First look for
 101 |     // implicit casts.
 102 |     if (!LHSCast || !RHSCast || !isImplicitCastCandidate(LHSCast) ||
 103 |         !isImplicitCastCandidate(RHSCast))
 104 |       continue;
 105 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const auto *LHSCast = dyn_cast<ImplicitCastExpr>(ignoreNoOpC`.
- CN: 这一段继续实现，围绕 `const auto *LHSCast = dyn_cast<ImplicitCastExpr>(ignoreNoOpC` 展开声明或语句。

### Lines 106-115
```cpp
 106 |     // If the types that go into the implicit casts match the types of the other
 107 |     // argument in the declaration there is a high probability that the
 108 |     // arguments were swapped.
 109 |     // TODO: We could make use of the edit distance between the argument name
 110 |     // and the name of the passed variable in addition to this type based
 111 |     // heuristic.
 112 |     const Expr *LHSFrom = ignoreNoOpCasts(LHSCast->getSubExpr());
 113 |     const Expr *RHSFrom = ignoreNoOpCasts(RHSCast->getSubExpr());
 114 |     if (!areArgumentsPotentiallySwapped(LHS->getType(), RHS->getType(),
 115 |                                         LHSFrom->getType(), RHSFrom->getType()))
```
- EN: This block continues the implementation with declarations or statements centered on `// If the types that go into the implicit casts match the ty`.
- CN: 这一段继续实现，围绕 `// If the types that go into the implicit casts match the ty` 展开声明或语句。

### Lines 116-125
```cpp
 116 |       continue;
 117 | 
 118 |     // Emit a warning and fix-its that swap the arguments.
 119 |     diag(Call->getBeginLoc(), "argument with implicit conversion from %0 "
 120 |                               "to %1 followed by argument converted from "
 121 |                               "%2 to %3, potentially swapped arguments.")
 122 |         << LHSFrom->getType() << LHS->getType() << RHSFrom->getType()
 123 |         << RHS->getType() << tooling::fixit::createReplacement(*LHS, *RHS, Ctx)
 124 |         << tooling::fixit::createReplacement(*RHS, *LHS, Ctx);
 125 | 
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

### Lines 126-131
```cpp
 126 |     // Remember that we emitted a warning for this argument.
 127 |     UsedArgs.insert(RHSCast);
 128 |   }
 129 | }
 130 | 
 131 | } // namespace clang::tidy::bugprone
```
- EN: This block continues the implementation with declarations or statements centered on `// Remember that we emitted a warning for this argument.`.
- CN: 这一段继续实现，围绕 `// Remember that we emitted a warning for this argument.` 展开声明或语句。

## Key Concepts / 关键概念
- AST matcher DSL / AST 匹配器 DSL
- diagnostics and fix-its / 诊断与修复建议
- namespace scoping / 命名空间作用域

## Dependencies / 依赖关系
- EN: Direct includes: `SwappedArgumentsCheck.h`, `clang/AST/ASTContext.h`, `clang/Lex/Lexer.h`, `clang/Tooling/FixIt.h`, `llvm/ADT/SmallPtrSet.h`.
- CN: 直接包含依赖: `SwappedArgumentsCheck.h`、`clang/AST/ASTContext.h`、`clang/Lex/Lexer.h`、`clang/Tooling/FixIt.h`、`llvm/ADT/SmallPtrSet.h`。
- EN: Namespace context: `clang::tidy::bugprone`.
- CN: 命名空间上下文: `clang::tidy::bugprone`。
