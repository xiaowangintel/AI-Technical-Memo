# StringConstructorCheck.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `clang-tools-extra/clang-tidy/bugprone/StringConstructorCheck.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `StringConstructorCheck` by registering AST matchers and producing diagnostics/fix-its.
- 用途 (CN): 通过注册 AST 匹配器并生成诊断/修复建议来实现 `StringConstructorCheck`。

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
   9 | #include "StringConstructorCheck.h"
  10 | #include "../utils/OptionsUtils.h"
  11 | #include "clang/AST/ASTContext.h"
  12 | #include "clang/ASTMatchers/ASTMatchFinder.h"
  13 | #include "clang/Tooling/FixIt.h"
  14 | 
```
- EN: The section imports dependencies such as `StringConstructorCheck.h`, `../utils/OptionsUtils.h`, `clang/AST/ASTContext.h`, `clang/ASTMatchers/ASTMatchFinder.h` needed by this file.
- CN: 本段引入了 `StringConstructorCheck.h`、`../utils/OptionsUtils.h`、`clang/AST/ASTContext.h`、`clang/ASTMatchers/ASTMatchFinder.h` 等依赖，供当前文件使用。

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
  20 | AST_MATCHER_P(IntegerLiteral, isBiggerThan, unsigned, N) {
  21 |   return Node.getValue().getZExtValue() > N;
  22 | }
  23 | } // namespace
  24 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 25-34
```cpp
  25 | static const char DefaultStringNames[] =
  26 |     "::std::basic_string;::std::basic_string_view";
  27 | 
  28 | static std::vector<StringRef>
  29 | removeNamespaces(const std::vector<StringRef> &Names) {
  30 |   std::vector<StringRef> Result;
  31 |   Result.reserve(Names.size());
  32 |   for (const StringRef Name : Names) {
  33 |     const std::string::size_type ColonPos = Name.rfind(':');
  34 |     Result.push_back(
```
- EN: This block continues the implementation with declarations or statements centered on `static const char DefaultStringNames[] =`.
- CN: 这一段继续实现，围绕 `static const char DefaultStringNames[] =` 展开声明或语句。

### Lines 35-39
```cpp
  35 |         Name.substr(ColonPos == std::string::npos ? 0 : ColonPos + 1));
  36 |   }
  37 |   return Result;
  38 | }
  39 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 40-49
```cpp
  40 | StringConstructorCheck::StringConstructorCheck(StringRef Name,
  41 |                                                ClangTidyContext *Context)
  42 |     : ClangTidyCheck(Name, Context),
  43 |       IsStringviewNullptrCheckEnabled(
  44 |           Context->isCheckEnabled("bugprone-stringview-nullptr")),
  45 |       WarnOnLargeLength(Options.get("WarnOnLargeLength", true)),
  46 |       LargeLengthThreshold(Options.get("LargeLengthThreshold", 0x800000)),
  47 |       StringNames(utils::options::parseStringList(
  48 |           Options.get("StringNames", DefaultStringNames))) {}
  49 | 
```
- EN: This block continues the implementation with declarations or statements centered on `StringConstructorCheck::StringConstructorCheck(StringRef Nam`.
- CN: 这一段继续实现，围绕 `StringConstructorCheck::StringConstructorCheck(StringRef Nam` 展开声明或语句。

### Lines 50-55
```cpp
  50 | void StringConstructorCheck::storeOptions(ClangTidyOptions::OptionMap &Opts) {
  51 |   Options.store(Opts, "WarnOnLargeLength", WarnOnLargeLength);
  52 |   Options.store(Opts, "LargeLengthThreshold", LargeLengthThreshold);
  53 |   Options.store(Opts, "StringNames", DefaultStringNames);
  54 | }
  55 | 
```
- EN: Method definitions such as `StringConstructorCheck::storeOptions` provide the concrete behavior declared elsewhere.
- CN: 诸如 `StringConstructorCheck::storeOptions` 的方法定义给出了前面声明的具体行为。

### Lines 56-65
```cpp
  56 | void StringConstructorCheck::registerMatchers(MatchFinder *Finder) {
  57 |   const auto ZeroExpr = expr(ignoringParenImpCasts(integerLiteral(equals(0))));
  58 |   const auto CharExpr = expr(ignoringParenImpCasts(characterLiteral()));
  59 |   const auto NegativeExpr = expr(ignoringParenImpCasts(
  60 |       unaryOperator(hasOperatorName("-"),
  61 |                     hasUnaryOperand(integerLiteral(unless(equals(0)))))));
  62 |   const auto LargeLengthExpr = expr(ignoringParenImpCasts(
  63 |       integerLiteral(isBiggerThan(LargeLengthThreshold))));
  64 |   const auto CharPtrType = type(anyOf(pointerType(), arrayType()));
  65 | 
```
- EN: This block defines AST matcher registration, i.e. the structural patterns the check will look for.
- CN: 这一段定义 AST 匹配器注册，也就是该检查要查找的结构模式。
- EN: Method definitions such as `StringConstructorCheck::registerMatchers` provide the concrete behavior declared elsewhere.
- CN: 诸如 `StringConstructorCheck::registerMatchers` 的方法定义给出了前面声明的具体行为。

### Lines 66-75
```cpp
  66 |   // Match a string-literal; even through a declaration with initializer.
  67 |   const auto BoundStringLiteral = stringLiteral().bind("str");
  68 |   const auto ConstStrLiteralDecl = varDecl(
  69 |       isDefinition(), hasType(constantArrayType()), hasType(isConstQualified()),
  70 |       hasInitializer(ignoringParenImpCasts(BoundStringLiteral)));
  71 |   const auto ConstPtrStrLiteralDecl = varDecl(
  72 |       isDefinition(),
  73 |       hasType(pointerType(pointee(isAnyCharacter(), isConstQualified()))),
  74 |       hasInitializer(ignoringParenImpCasts(BoundStringLiteral)));
  75 |   const auto ConstStrLiteral = expr(ignoringParenImpCasts(anyOf(
```
- EN: This block continues the implementation with declarations or statements centered on `// Match a string-literal; even through a declaration with i`.
- CN: 这一段继续实现，围绕 `// Match a string-literal; even through a declaration with i` 展开声明或语句。

### Lines 76-85
```cpp
  76 |       BoundStringLiteral, declRefExpr(hasDeclaration(anyOf(
  77 |                               ConstPtrStrLiteralDecl, ConstStrLiteralDecl))))));
  78 | 
  79 |   // Check the fill constructor. Fills the string with n consecutive copies of
  80 |   // character c. [i.e string(size_t n, char c);].
  81 |   Finder->addMatcher(
  82 |       cxxConstructExpr(
  83 |           hasDeclaration(cxxMethodDecl(hasName("basic_string"))),
  84 |           anyOf(argumentCountIs(2), argumentCountIs(3)),
  85 |           hasArgument(0, hasType(qualType(isInteger()))),
```
- EN: This block continues the implementation with declarations or statements centered on `BoundStringLiteral, declRefExpr(hasDeclaration(anyOf(`.
- CN: 这一段继续实现，围绕 `BoundStringLiteral, declRefExpr(hasDeclaration(anyOf(` 展开声明或语句。

### Lines 86-95
```cpp
  86 |           hasArgument(1, hasType(qualType(isInteger()))),
  87 |           anyOf(
  88 |               // Detect the expression: string('x', 40);
  89 |               hasArgument(0, CharExpr.bind("swapped-parameter")),
  90 |               // Detect the expression: string(0, ...);
  91 |               hasArgument(0, ZeroExpr.bind("empty-string")),
  92 |               // Detect the expression: string(-4, ...);
  93 |               hasArgument(0, NegativeExpr.bind("negative-length")),
  94 |               // Detect the expression: string(0x1234567, ...);
  95 |               hasArgument(0, LargeLengthExpr.bind("large-length"))))
```
- EN: This block continues the implementation with declarations or statements centered on `hasArgument(1, hasType(qualType(isInteger()))),`.
- CN: 这一段继续实现，围绕 `hasArgument(1, hasType(qualType(isInteger()))),` 展开声明或语句。

### Lines 96-105
```cpp
  96 |           .bind("constructor"),
  97 |       this);
  98 | 
  99 |   // Check the literal string constructor with char pointer and length
 100 |   // parameters. [i.e. string (const char* s, size_t n);]
 101 |   Finder->addMatcher(
 102 |       cxxConstructExpr(
 103 |           hasDeclaration(cxxConstructorDecl(ofClass(
 104 |               cxxRecordDecl(hasAnyName(removeNamespaces(StringNames)))))),
 105 |           anyOf(argumentCountIs(2),
```
- EN: This block continues the implementation with declarations or statements centered on `.bind("constructor"),`.
- CN: 这一段继续实现，围绕 `.bind("constructor"),` 展开声明或语句。

### Lines 106-115
```cpp
 106 |                 allOf(argumentCountIs(3),
 107 |                       hasArgument(2, unless(hasType(qualType(isInteger())))))),
 108 |           hasArgument(0, hasType(CharPtrType)),
 109 |           hasArgument(1, hasType(isInteger())),
 110 |           anyOf(
 111 |               // Detect the expression: string("...", 0);
 112 |               hasArgument(1, ZeroExpr.bind("empty-string")),
 113 |               // Detect the expression: string("...", -4);
 114 |               hasArgument(1, NegativeExpr.bind("negative-length")),
 115 |               // Detect the expression: string("lit", 0x1234567);
```
- EN: This block continues the implementation with declarations or statements centered on `allOf(argumentCountIs(3),`.
- CN: 这一段继续实现，围绕 `allOf(argumentCountIs(3),` 展开声明或语句。

### Lines 116-123
```cpp
 116 |               hasArgument(1, LargeLengthExpr.bind("large-length")),
 117 |               // Detect the expression: string("lit", 5)
 118 |               allOf(hasArgument(0, ConstStrLiteral.bind("literal-with-length")),
 119 |                     hasArgument(1, ignoringParenImpCasts(
 120 |                                        integerLiteral().bind("length"))))))
 121 |           .bind("constructor"),
 122 |       this);
 123 | 
```
- EN: This block continues the implementation with declarations or statements centered on `hasArgument(1, LargeLengthExpr.bind("large-length")),`.
- CN: 这一段继续实现，围绕 `hasArgument(1, LargeLengthExpr.bind("large-length")),` 展开声明或语句。

### Lines 124-133
```cpp
 124 |   // Check the literal string constructor with char pointer, start position and
 125 |   // length parameters. [i.e. string (const char* s, size_t pos, size_t count);]
 126 |   Finder->addMatcher(
 127 |       cxxConstructExpr(
 128 |           hasDeclaration(cxxConstructorDecl(ofClass(
 129 |               cxxRecordDecl(hasAnyName(removeNamespaces(StringNames)))))),
 130 |           anyOf(argumentCountIs(3), argumentCountIs(4)),
 131 |           hasArgument(0, hasType(CharPtrType)),
 132 |           hasArgument(1, hasType(qualType(isInteger()))),
 133 |           hasArgument(2, hasType(qualType(isInteger()))),
```
- EN: This block continues the implementation with declarations or statements centered on `// Check the literal string constructor with char pointer, s`.
- CN: 这一段继续实现，围绕 `// Check the literal string constructor with char pointer, s` 展开声明或语句。

### Lines 134-143
```cpp
 134 |           anyOf(
 135 |               // Detect the expression: string("...", 1, 0);
 136 |               hasArgument(2, ZeroExpr.bind("empty-string")),
 137 |               // Detect the expression: string("...", -4, 1);
 138 |               hasArgument(1, NegativeExpr.bind("negative-pos")),
 139 |               // Detect the expression: string("...", 0, -4);
 140 |               hasArgument(2, NegativeExpr.bind("negative-length")),
 141 |               // Detect the expression: string("lit", 0, 0x1234567);
 142 |               hasArgument(2, LargeLengthExpr.bind("large-length")),
 143 |               // Detect the expression: string("lit", 1, 5)
```
- EN: This block continues the implementation with declarations or statements centered on `anyOf(`.
- CN: 这一段继续实现，围绕 `anyOf(` 展开声明或语句。

### Lines 144-151
```cpp
 144 |               allOf(hasArgument(0, ConstStrLiteral.bind("literal-with-length")),
 145 |                     hasArgument(
 146 |                         1, ignoringParenImpCasts(integerLiteral().bind("pos"))),
 147 |                     hasArgument(2, ignoringParenImpCasts(
 148 |                                        integerLiteral().bind("length"))))))
 149 |           .bind("constructor"),
 150 |       this);
 151 | 
```
- EN: This block continues the implementation with declarations or statements centered on `allOf(hasArgument(0, ConstStrLiteral.bind("literal-with-leng`.
- CN: 这一段继续实现，围绕 `allOf(hasArgument(0, ConstStrLiteral.bind("literal-with-leng` 展开声明或语句。

### Lines 152-161
```cpp
 152 |   // Check the literal string constructor with char pointer.
 153 |   // [i.e. string (const char* s);]
 154 |   Finder->addMatcher(
 155 |       traverse(
 156 |           TK_AsIs,
 157 |           cxxConstructExpr(
 158 |               hasDeclaration(cxxConstructorDecl(ofClass(anyOf(
 159 |                   cxxRecordDecl(hasName("basic_string_view"))
 160 |                       .bind("basic_string_view_decl"),
 161 |                   cxxRecordDecl(hasAnyName(removeNamespaces(StringNames))))))),
```
- EN: This block continues the implementation with declarations or statements centered on `// Check the literal string constructor with char pointer.`.
- CN: 这一段继续实现，围绕 `// Check the literal string constructor with char pointer.` 展开声明或语句。

### Lines 162-171
```cpp
 162 |               hasArgument(0, expr().bind("from-ptr")),
 163 |               // do not match std::string(ptr, int)
 164 |               // match std::string(ptr, alloc)
 165 |               // match std::string(ptr)
 166 |               anyOf(hasArgument(1, unless(hasType(isInteger()))),
 167 |                     argumentCountIs(1)))
 168 |               .bind("constructor")),
 169 |       this);
 170 | }
 171 | 
```
- EN: This block continues the implementation with declarations or statements centered on `hasArgument(0, expr().bind("from-ptr")),`.
- CN: 这一段继续实现，围绕 `hasArgument(0, expr().bind("from-ptr")),` 展开声明或语句。

### Lines 172-177
```cpp
 172 | void StringConstructorCheck::check(const MatchFinder::MatchResult &Result) {
 173 |   const ASTContext &Ctx = *Result.Context;
 174 |   const auto *E = Result.Nodes.getNodeAs<CXXConstructExpr>("constructor");
 175 |   assert(E && "missing constructor expression");
 176 |   const SourceLocation Loc = E->getBeginLoc();
 177 | 
```
- EN: Method definitions such as `StringConstructorCheck::check` provide the concrete behavior declared elsewhere.
- CN: 诸如 `StringConstructorCheck::check` 的方法定义给出了前面声明的具体行为。

### Lines 178-187
```cpp
 178 |   if (Result.Nodes.getNodeAs<Expr>("swapped-parameter")) {
 179 |     const Expr *P0 = E->getArg(0);
 180 |     const Expr *P1 = E->getArg(1);
 181 |     diag(Loc, "string constructor parameters are probably swapped;"
 182 |               " expecting string(count, character)")
 183 |         << tooling::fixit::createReplacement(*P0, *P1, Ctx)
 184 |         << tooling::fixit::createReplacement(*P1, *P0, Ctx);
 185 |   } else if (Result.Nodes.getNodeAs<Expr>("empty-string")) {
 186 |     diag(Loc, "constructor creating an empty string");
 187 |   } else if (Result.Nodes.getNodeAs<Expr>("negative-length")) {
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

### Lines 188-197
```cpp
 188 |     diag(Loc, "negative value used as length parameter");
 189 |   } else if (Result.Nodes.getNodeAs<Expr>("negative-pos")) {
 190 |     diag(Loc, "negative value used as position of the "
 191 |               "first character parameter");
 192 |   } else if (Result.Nodes.getNodeAs<Expr>("large-length")) {
 193 |     if (WarnOnLargeLength)
 194 |       diag(Loc, "suspicious large length parameter");
 195 |   } else if (Result.Nodes.getNodeAs<Expr>("literal-with-length")) {
 196 |     const auto *Str = Result.Nodes.getNodeAs<StringLiteral>("str");
 197 |     const auto *Length = Result.Nodes.getNodeAs<IntegerLiteral>("length");
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

### Lines 198-207
```cpp
 198 |     if (Length->getValue().ugt(Str->getLength())) {
 199 |       diag(Loc, "length is bigger than string literal size");
 200 |       return;
 201 |     }
 202 |     if (const auto *Pos = Result.Nodes.getNodeAs<IntegerLiteral>("pos")) {
 203 |       if (Pos->getValue().uge(Str->getLength())) {
 204 |         diag(Loc, "position of the first character parameter is bigger than "
 205 |                   "string literal character range");
 206 |       } else if (Length->getValue().ugt(
 207 |                      (Str->getLength() - Pos->getValue()).getZExtValue())) {
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

### Lines 208-217
```cpp
 208 |         diag(Loc, "length is bigger than remaining string literal size");
 209 |       }
 210 |     }
 211 |   } else if (const auto *Ptr = Result.Nodes.getNodeAs<Expr>("from-ptr")) {
 212 |     Expr::EvalResult ConstPtr;
 213 |     if (!Ptr->isInstantiationDependent() &&
 214 |         Ptr->EvaluateAsRValue(ConstPtr, Ctx) &&
 215 |         ((ConstPtr.Val.isInt() && ConstPtr.Val.getInt().isZero()) ||
 216 |          (ConstPtr.Val.isLValue() && ConstPtr.Val.isNullPointer()))) {
 217 |       if (IsStringviewNullptrCheckEnabled &&
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

### Lines 218-228
```cpp
 218 |           Result.Nodes.getNodeAs<CXXRecordDecl>("basic_string_view_decl")) {
 219 |         // Filter out `basic_string_view` to avoid conflicts with
 220 |         // `bugprone-stringview-nullptr`
 221 |         return;
 222 |       }
 223 |       diag(Loc, "constructing string from nullptr is undefined behaviour");
 224 |     }
 225 |   }
 226 | }
 227 | 
 228 | } // namespace clang::tidy::bugprone
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

## Key Concepts / 关键概念
- clang-tidy check lifecycle / clang-tidy 检查生命周期
- AST matcher DSL / AST 匹配器 DSL
- diagnostics and fix-its / 诊断与修复建议
- namespace scoping / 命名空间作用域

## Dependencies / 依赖关系
- EN: Direct includes: `StringConstructorCheck.h`, `../utils/OptionsUtils.h`, `clang/AST/ASTContext.h`, `clang/ASTMatchers/ASTMatchFinder.h`, `clang/Tooling/FixIt.h`.
- CN: 直接包含依赖: `StringConstructorCheck.h`、`../utils/OptionsUtils.h`、`clang/AST/ASTContext.h`、`clang/ASTMatchers/ASTMatchFinder.h`、`clang/Tooling/FixIt.h`。
- EN: Namespace context: `clang::tidy::bugprone`.
- CN: 命名空间上下文: `clang::tidy::bugprone`。
