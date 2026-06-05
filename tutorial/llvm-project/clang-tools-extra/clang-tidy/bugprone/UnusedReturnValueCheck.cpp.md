# UnusedReturnValueCheck.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `clang-tools-extra/clang-tidy/bugprone/UnusedReturnValueCheck.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `llvm` by registering AST matchers and producing diagnostics/fix-its.
- 用途 (CN): 通过注册 AST 匹配器并生成诊断/修复建议来实现 `llvm`。

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
   9 | #include "UnusedReturnValueCheck.h"
  10 | #include "../utils/Matchers.h"
  11 | #include "../utils/OptionsUtils.h"
  12 | #include "clang/ASTMatchers/ASTMatchFinder.h"
  13 | #include "clang/ASTMatchers/ASTMatchers.h"
  14 | #include "clang/Basic/OperatorKinds.h"
  15 | 
```
- EN: The section imports dependencies such as `UnusedReturnValueCheck.h`, `../utils/Matchers.h`, `../utils/OptionsUtils.h`, `clang/ASTMatchers/ASTMatchFinder.h` needed by this file.
- CN: 本段引入了 `UnusedReturnValueCheck.h`、`../utils/Matchers.h`、`../utils/OptionsUtils.h`、`clang/ASTMatchers/ASTMatchFinder.h` 等依赖，供当前文件使用。

### Lines 16-20
```cpp
  16 | using namespace clang::ast_matchers;
  17 | using namespace clang::ast_matchers::internal;
  18 | 
  19 | namespace clang::tidy::bugprone {
  20 | 
```
- EN: Using-directives shorten qualified names so matcher/transformer code reads more directly.
- CN: using 指令缩短了限定名，使匹配器/变换器代码更直接。
- EN: Namespace scopes such as `clang::tidy::bugprone` place the symbols in their intended subsystem.
- CN: 诸如 `clang::tidy::bugprone` 这样的命名空间将符号放入预期的子系统中。

### Lines 21-30
```cpp
  21 | namespace {
  22 | 
  23 | // Matches functions that are instantiated from a class template member function
  24 | // matching InnerMatcher. Functions not instantiated from a class template
  25 | // member function are matched directly with InnerMatcher.
  26 | AST_MATCHER_P(FunctionDecl, isInstantiatedFrom, Matcher<FunctionDecl>,
  27 |               InnerMatcher) {
  28 |   const FunctionDecl *InstantiatedFrom =
  29 |       Node.getInstantiatedFromMemberFunction();
  30 |   return InnerMatcher.matches(InstantiatedFrom ? *InstantiatedFrom : Node,
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 31-40
```cpp
  31 |                               Finder, Builder);
  32 | }
  33 | 
  34 | constexpr std::initializer_list<OverloadedOperatorKind>
  35 |     AssignmentOverloadedOperatorKinds = {
  36 |         OO_Equal,      OO_PlusEqual,     OO_MinusEqual,          OO_StarEqual,
  37 |         OO_SlashEqual, OO_PercentEqual,  OO_CaretEqual,          OO_AmpEqual,
  38 |         OO_PipeEqual,  OO_LessLessEqual, OO_GreaterGreaterEqual, OO_PlusPlus,
  39 |         OO_MinusMinus};
  40 | 
```
- EN: This block continues the implementation with declarations or statements centered on `Finder, Builder);`.
- CN: 这一段继续实现，围绕 `Finder, Builder);` 展开声明或语句。

### Lines 41-46
```cpp
  41 | AST_MATCHER(FunctionDecl, isAssignmentOverloadedOperator) {
  42 |   return llvm::is_contained(AssignmentOverloadedOperatorKinds,
  43 |                             Node.getOverloadedOperator());
  44 | }
  45 | } // namespace
  46 | 
```
- EN: Method definitions such as `llvm::is_contained` provide the concrete behavior declared elsewhere.
- CN: 诸如 `llvm::is_contained` 的方法定义给出了前面声明的具体行为。
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 47-56
```cpp
  47 | UnusedReturnValueCheck::UnusedReturnValueCheck(StringRef Name,
  48 |                                                ClangTidyContext *Context)
  49 |     : ClangTidyCheck(Name, Context),
  50 |       CheckedFunctions(utils::options::parseStringList(
  51 |           Options.get("CheckedFunctions", "^::std::async$;"
  52 |                                           "^::std::launder$;"
  53 |                                           "^::std::remove$;"
  54 |                                           "^::std::remove_if$;"
  55 |                                           "^::std::unique$;"
  56 |                                           "^::std::unique_ptr::release$;"
```
- EN: This block continues the implementation with declarations or statements centered on `UnusedReturnValueCheck::UnusedReturnValueCheck(StringRef Nam`.
- CN: 这一段继续实现，围绕 `UnusedReturnValueCheck::UnusedReturnValueCheck(StringRef Nam` 展开声明或语句。

### Lines 57-66
```cpp
  57 |                                           "^::std::basic_string::empty$;"
  58 |                                           "^::std::vector::empty$;"
  59 |                                           "^::std::back_inserter$;"
  60 |                                           "^::std::distance$;"
  61 |                                           "^::std::find$;"
  62 |                                           "^::std::find_if$;"
  63 |                                           "^::std::inserter$;"
  64 |                                           "^::std::lower_bound$;"
  65 |                                           "^::std::make_pair$;"
  66 |                                           "^::std::map::count$;"
```
- EN: This block continues the implementation with declarations or statements centered on `"^::std::basic_string::empty$;"`.
- CN: 这一段继续实现，围绕 `"^::std::basic_string::empty$;"` 展开声明或语句。

### Lines 67-76
```cpp
  67 |                                           "^::std::map::find$;"
  68 |                                           "^::std::map::lower_bound$;"
  69 |                                           "^::std::multimap::equal_range$;"
  70 |                                           "^::std::multimap::upper_bound$;"
  71 |                                           "^::std::set::count$;"
  72 |                                           "^::std::set::find$;"
  73 |                                           "^::std::setfill$;"
  74 |                                           "^::std::setprecision$;"
  75 |                                           "^::std::setw$;"
  76 |                                           "^::std::upper_bound$;"
```
- EN: This block continues the implementation with declarations or statements centered on `"^::std::map::find$;"`.
- CN: 这一段继续实现，围绕 `"^::std::map::find$;"` 展开声明或语句。

### Lines 77-86
```cpp
  77 |                                           "^::std::vector::at$;"
  78 |                                           // C standard library
  79 |                                           "^::bsearch$;"
  80 |                                           "^::ferror$;"
  81 |                                           "^::feof$;"
  82 |                                           "^::isalnum$;"
  83 |                                           "^::isalpha$;"
  84 |                                           "^::isblank$;"
  85 |                                           "^::iscntrl$;"
  86 |                                           "^::isdigit$;"
```
- EN: This block continues the implementation with declarations or statements centered on `"^::std::vector::at$;"`.
- CN: 这一段继续实现，围绕 `"^::std::vector::at$;"` 展开声明或语句。

### Lines 87-96
```cpp
  87 |                                           "^::isgraph$;"
  88 |                                           "^::islower$;"
  89 |                                           "^::isprint$;"
  90 |                                           "^::ispunct$;"
  91 |                                           "^::isspace$;"
  92 |                                           "^::isupper$;"
  93 |                                           "^::iswalnum$;"
  94 |                                           "^::iswprint$;"
  95 |                                           "^::iswspace$;"
  96 |                                           "^::isxdigit$;"
```
- EN: This block continues the implementation with declarations or statements centered on `"^::isgraph$;"`.
- CN: 这一段继续实现，围绕 `"^::isgraph$;"` 展开声明或语句。

### Lines 97-106
```cpp
  97 |                                           "^::memchr$;"
  98 |                                           "^::memcmp$;"
  99 |                                           "^::strcmp$;"
 100 |                                           "^::strcoll$;"
 101 |                                           "^::strncmp$;"
 102 |                                           "^::strpbrk$;"
 103 |                                           "^::strrchr$;"
 104 |                                           "^::strspn$;"
 105 |                                           "^::strstr$;"
 106 |                                           "^::wcscmp$;"
```
- EN: This block continues the implementation with declarations or statements centered on `"^::memchr$;"`.
- CN: 这一段继续实现，围绕 `"^::memchr$;"` 展开声明或语句。

### Lines 107-116
```cpp
 107 |                                           // POSIX
 108 |                                           "^::access$;"
 109 |                                           "^::bind$;"
 110 |                                           "^::connect$;"
 111 |                                           "^::difftime$;"
 112 |                                           "^::dlsym$;"
 113 |                                           "^::fnmatch$;"
 114 |                                           "^::getaddrinfo$;"
 115 |                                           "^::getopt$;"
 116 |                                           "^::htonl$;"
```
- EN: This block continues the implementation with declarations or statements centered on `// POSIX`.
- CN: 这一段继续实现，围绕 `// POSIX` 展开声明或语句。

### Lines 117-126
```cpp
 117 |                                           "^::htons$;"
 118 |                                           "^::iconv_open$;"
 119 |                                           "^::inet_addr$;"
 120 |                                           "^::isascii$;"
 121 |                                           "^::isatty$;"
 122 |                                           "^::mmap$;"
 123 |                                           "^::newlocale$;"
 124 |                                           "^::openat$;"
 125 |                                           "^::pathconf$;"
 126 |                                           "^::pthread_equal$;"
```
- EN: This block continues the implementation with declarations or statements centered on `"^::htons$;"`.
- CN: 这一段继续实现，围绕 `"^::htons$;"` 展开声明或语句。

### Lines 127-136
```cpp
 127 |                                           "^::pthread_getspecific$;"
 128 |                                           "^::pthread_mutex_trylock$;"
 129 |                                           "^::readdir$;"
 130 |                                           "^::readlink$;"
 131 |                                           "^::recvmsg$;"
 132 |                                           "^::regexec$;"
 133 |                                           "^::scandir$;"
 134 |                                           "^::semget$;"
 135 |                                           "^::setjmp$;"
 136 |                                           "^::shm_open$;"
```
- EN: This block continues the implementation with declarations or statements centered on `"^::pthread_getspecific$;"`.
- CN: 这一段继续实现，围绕 `"^::pthread_getspecific$;"` 展开声明或语句。

### Lines 137-146
```cpp
 137 |                                           "^::shmget$;"
 138 |                                           "^::sigismember$;"
 139 |                                           "^::strcasecmp$;"
 140 |                                           "^::strsignal$;"
 141 |                                           "^::ttyname$"))),
 142 |       CheckedReturnTypes(utils::options::parseStringList(
 143 |           Options.get("CheckedReturnTypes", "^::std::error_code$;"
 144 |                                             "^::std::error_condition$;"
 145 |                                             "^::std::errc$;"
 146 |                                             "^::std::expected$;"
```
- EN: This block continues the implementation with declarations or statements centered on `"^::shmget$;"`.
- CN: 这一段继续实现，围绕 `"^::shmget$;"` 展开声明或语句。

### Lines 147-155
```cpp
 147 |                                             "^::boost::system::error_code$"))),
 148 |       AllowCastToVoid(Options.get("AllowCastToVoid", false)) {}
 149 | 
 150 | UnusedReturnValueCheck::UnusedReturnValueCheck(
 151 |     StringRef Name, ClangTidyContext *Context,
 152 |     std::vector<StringRef> CheckedFunctions)
 153 |     : UnusedReturnValueCheck(Name, Context, std::move(CheckedFunctions), {},
 154 |                              false) {}
 155 | 
```
- EN: Method definitions such as `UnusedReturnValueCheck::UnusedReturnValueCheck` provide the concrete behavior declared elsewhere.
- CN: 诸如 `UnusedReturnValueCheck::UnusedReturnValueCheck` 的方法定义给出了前面声明的具体行为。

### Lines 156-164
```cpp
 156 | UnusedReturnValueCheck::UnusedReturnValueCheck(
 157 |     StringRef Name, ClangTidyContext *Context,
 158 |     std::vector<StringRef> CheckedFunctions,
 159 |     std::vector<StringRef> CheckedReturnTypes, bool AllowCastToVoid)
 160 |     : ClangTidyCheck(Name, Context),
 161 |       CheckedFunctions(std::move(CheckedFunctions)),
 162 |       CheckedReturnTypes(std::move(CheckedReturnTypes)),
 163 |       AllowCastToVoid(AllowCastToVoid) {}
 164 | 
```
- EN: This block continues the implementation with declarations or statements centered on `UnusedReturnValueCheck::UnusedReturnValueCheck(`.
- CN: 这一段继续实现，围绕 `UnusedReturnValueCheck::UnusedReturnValueCheck(` 展开声明或语句。

### Lines 165-172
```cpp
 165 | void UnusedReturnValueCheck::storeOptions(ClangTidyOptions::OptionMap &Opts) {
 166 |   Options.store(Opts, "CheckedFunctions",
 167 |                 utils::options::serializeStringList(CheckedFunctions));
 168 |   Options.store(Opts, "CheckedReturnTypes",
 169 |                 utils::options::serializeStringList(CheckedReturnTypes));
 170 |   Options.store(Opts, "AllowCastToVoid", AllowCastToVoid);
 171 | }
 172 | 
```
- EN: Method definitions such as `UnusedReturnValueCheck::storeOptions` provide the concrete behavior declared elsewhere.
- CN: 诸如 `UnusedReturnValueCheck::storeOptions` 的方法定义给出了前面声明的具体行为。

### Lines 173-182
```cpp
 173 | void UnusedReturnValueCheck::registerMatchers(MatchFinder *Finder) {
 174 |   auto MatchedDirectCallExpr = expr(
 175 |       callExpr(callee(functionDecl(
 176 |                    // Don't match copy or move assignment operator.
 177 |                    unless(isAssignmentOverloadedOperator()),
 178 |                    // Don't match void overloads of checked functions.
 179 |                    unless(returns(voidType())),
 180 |                    anyOf(isInstantiatedFrom(matchers::matchesAnyListedRegexName(
 181 |                              CheckedFunctions)),
 182 |                          returns(hasCanonicalType(hasDeclaration(
```
- EN: This block defines AST matcher registration, i.e. the structural patterns the check will look for.
- CN: 这一段定义 AST 匹配器注册，也就是该检查要查找的结构模式。
- EN: Method definitions such as `UnusedReturnValueCheck::registerMatchers` provide the concrete behavior declared elsewhere.
- CN: 诸如 `UnusedReturnValueCheck::registerMatchers` 的方法定义给出了前面声明的具体行为。

### Lines 183-186
```cpp
 183 |                              namedDecl(matchers::matchesAnyListedRegexName(
 184 |                                  CheckedReturnTypes)))))))))
 185 |           .bind("match"));
 186 | 
```
- EN: This block continues the implementation with declarations or statements centered on `namedDecl(matchers::matchesAnyListedRegexName(`.
- CN: 这一段继续实现，围绕 `namedDecl(matchers::matchesAnyListedRegexName(` 展开声明或语句。

### Lines 187-193
```cpp
 187 |   auto CheckCastToVoid =
 188 |       AllowCastToVoid ? castExpr(unless(hasCastKind(CK_ToVoid))) : castExpr();
 189 |   auto MatchedCallExpr = expr(
 190 |       anyOf(MatchedDirectCallExpr,
 191 |             explicitCastExpr(unless(cxxFunctionalCastExpr()), CheckCastToVoid,
 192 |                              hasSourceExpression(MatchedDirectCallExpr))));
 193 | 
```
- EN: This block continues the implementation with declarations or statements centered on `auto CheckCastToVoid =`.
- CN: 这一段继续实现，围绕 `auto CheckCastToVoid =` 展开声明或语句。

### Lines 194-203
```cpp
 194 |   auto UnusedInCompoundStmt =
 195 |       compoundStmt(forEach(MatchedCallExpr),
 196 |                    // The checker can't currently differentiate between the
 197 |                    // return statement and other statements inside GNU statement
 198 |                    // expressions, so disable the checker inside them to avoid
 199 |                    // false positives.
 200 |                    unless(hasParent(stmtExpr())));
 201 |   auto UnusedInIfStmt =
 202 |       ifStmt(eachOf(hasThen(MatchedCallExpr), hasElse(MatchedCallExpr)));
 203 |   auto UnusedInWhileStmt = whileStmt(hasBody(MatchedCallExpr));
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 204-210
```cpp
 204 |   auto UnusedInDoStmt = doStmt(hasBody(MatchedCallExpr));
 205 |   auto UnusedInForStmt =
 206 |       forStmt(eachOf(hasLoopInit(MatchedCallExpr),
 207 |                      hasIncrement(MatchedCallExpr), hasBody(MatchedCallExpr)));
 208 |   auto UnusedInRangeForStmt = cxxForRangeStmt(hasBody(MatchedCallExpr));
 209 |   auto UnusedInCaseStmt = switchCase(forEach(MatchedCallExpr));
 210 | 
```
- EN: This block continues the implementation with declarations or statements centered on `auto UnusedInDoStmt = doStmt(hasBody(MatchedCallExpr));`.
- CN: 这一段继续实现，围绕 `auto UnusedInDoStmt = doStmt(hasBody(MatchedCallExpr));` 展开声明或语句。

### Lines 211-217
```cpp
 211 |   Finder->addMatcher(
 212 |       stmt(anyOf(UnusedInCompoundStmt, UnusedInIfStmt, UnusedInWhileStmt,
 213 |                  UnusedInDoStmt, UnusedInForStmt, UnusedInRangeForStmt,
 214 |                  UnusedInCaseStmt)),
 215 |       this);
 216 | }
 217 | 
```
- EN: This block continues the implementation with declarations or statements centered on `Finder->addMatcher(`.
- CN: 这一段继续实现，围绕 `Finder->addMatcher(` 展开声明或语句。

### Lines 218-224
```cpp
 218 | void UnusedReturnValueCheck::check(const MatchFinder::MatchResult &Result) {
 219 |   if (const auto *Matched = Result.Nodes.getNodeAs<CallExpr>("match")) {
 220 |     diag(Matched->getBeginLoc(),
 221 |          "the value returned by this function should not be disregarded; "
 222 |          "neglecting it may lead to errors")
 223 |         << Matched->getSourceRange();
 224 | 
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。
- EN: Method definitions such as `UnusedReturnValueCheck::check` provide the concrete behavior declared elsewhere.
- CN: 诸如 `UnusedReturnValueCheck::check` 的方法定义给出了前面声明的具体行为。

### Lines 225-234
```cpp
 225 |     if (!AllowCastToVoid)
 226 |       return;
 227 | 
 228 |     diag(Matched->getBeginLoc(),
 229 |          "cast the expression to void to silence this warning",
 230 |          DiagnosticIDs::Note);
 231 |   }
 232 | }
 233 | 
 234 | } // namespace clang::tidy::bugprone
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

## Key Concepts / 关键概念
- clang-tidy check lifecycle / clang-tidy 检查生命周期
- AST matcher DSL / AST 匹配器 DSL
- diagnostics and fix-its / 诊断与修复建议
- namespace scoping / 命名空间作用域

## Dependencies / 依赖关系
- EN: Direct includes: `UnusedReturnValueCheck.h`, `../utils/Matchers.h`, `../utils/OptionsUtils.h`, `clang/ASTMatchers/ASTMatchFinder.h`, `clang/ASTMatchers/ASTMatchers.h`, `clang/Basic/OperatorKinds.h`.
- CN: 直接包含依赖: `UnusedReturnValueCheck.h`、`../utils/Matchers.h`、`../utils/OptionsUtils.h`、`clang/ASTMatchers/ASTMatchFinder.h`、`clang/ASTMatchers/ASTMatchers.h`、`clang/Basic/OperatorKinds.h`。
- EN: Namespace context: `clang::tidy::bugprone`.
- CN: 命名空间上下文: `clang::tidy::bugprone`。
