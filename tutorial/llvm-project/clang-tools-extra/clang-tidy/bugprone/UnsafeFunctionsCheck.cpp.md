# UnsafeFunctionsCheck.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `clang-tools-extra/clang-tidy/bugprone/UnsafeFunctionsCheck.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `options` by registering AST matchers and producing diagnostics/fix-its.
- 用途 (CN): 通过注册 AST 匹配器并生成诊断/修复建议来实现 `options`。

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

### Lines 9-16
```cpp
   9 | #include "UnsafeFunctionsCheck.h"
  10 | #include "../utils/OptionsUtils.h"
  11 | #include "clang/AST/ASTContext.h"
  12 | #include "clang/ASTMatchers/ASTMatchFinder.h"
  13 | #include "clang/Analysis/AnnexKDetection.h"
  14 | #include "clang/Lex/Preprocessor.h"
  15 | #include <cassert>
  16 | 
```
- EN: The section imports dependencies such as `UnsafeFunctionsCheck.h`, `../utils/OptionsUtils.h`, `clang/AST/ASTContext.h`, `clang/ASTMatchers/ASTMatchFinder.h` needed by this file.
- CN: 本段引入了 `UnsafeFunctionsCheck.h`、`../utils/OptionsUtils.h`、`clang/AST/ASTContext.h`、`clang/ASTMatchers/ASTMatchFinder.h` 等依赖，供当前文件使用。

### Lines 17-21
```cpp
  17 | using namespace clang::ast_matchers;
  18 | using namespace llvm;
  19 | 
  20 | namespace clang::tidy::bugprone {
  21 | 
```
- EN: Using-directives shorten qualified names so matcher/transformer code reads more directly.
- CN: using 指令缩短了限定名，使匹配器/变换器代码更直接。
- EN: Namespace scopes such as `clang::tidy::bugprone` place the symbols in their intended subsystem.
- CN: 诸如 `clang::tidy::bugprone` 这样的命名空间将符号放入预期的子系统中。

### Lines 22-27
```cpp
  22 | static constexpr StringRef OptionNameCustomFunctions = "CustomFunctions";
  23 | static constexpr StringRef OptionNameReportDefaultFunctions =
  24 |     "ReportDefaultFunctions";
  25 | static constexpr StringRef OptionNameReportMoreUnsafeFunctions =
  26 |     "ReportMoreUnsafeFunctions";
  27 | 
```
- EN: This block continues the implementation with declarations or statements centered on `static constexpr StringRef OptionNameCustomFunctions = "Cust`.
- CN: 这一段继续实现，围绕 `static constexpr StringRef OptionNameCustomFunctions = "Cust` 展开声明或语句。

### Lines 28-35
```cpp
  28 | static constexpr StringRef FunctionNamesWithAnnexKReplacementId =
  29 |     "FunctionNamesWithAnnexKReplacement";
  30 | static constexpr StringRef FunctionNamesId = "FunctionsNames";
  31 | static constexpr StringRef AdditionalFunctionNamesId =
  32 |     "AdditionalFunctionsNames";
  33 | static constexpr StringRef CustomFunctionNamesId = "CustomFunctionNames";
  34 | static constexpr StringRef DeclRefId = "DRE";
  35 | 
```
- EN: This block continues the implementation with declarations or statements centered on `static constexpr StringRef FunctionNamesWithAnnexKReplacemen`.
- CN: 这一段继续实现，围绕 `static constexpr StringRef FunctionNamesWithAnnexKReplacemen` 展开声明或语句。

### Lines 36-43
```cpp
  36 | static std::optional<std::string>
  37 | getAnnexKReplacementFor(StringRef FunctionName) {
  38 |   return StringSwitch<std::string>(FunctionName)
  39 |       .Case("strlen", "strnlen_s")
  40 |       .Case("wcslen", "wcsnlen_s")
  41 |       .Default((Twine{FunctionName} + "_s").str());
  42 | }
  43 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 44-53
```cpp
  44 | static StringRef getReplacementFor(StringRef FunctionName,
  45 |                                    bool IsAnnexKAvailable) {
  46 |   if (IsAnnexKAvailable) {
  47 |     // Try to find a better replacement from Annex K first.
  48 |     StringRef AnnexKReplacementFunction =
  49 |         StringSwitch<StringRef>(FunctionName)
  50 |             .Cases({"asctime", "asctime_r"}, "asctime_s")
  51 |             .Case("gets", "gets_s")
  52 |             .Default({});
  53 |     if (!AnnexKReplacementFunction.empty())
```
- EN: This block continues the implementation with declarations or statements centered on `static StringRef getReplacementFor(StringRef FunctionName,`.
- CN: 这一段继续实现，围绕 `static StringRef getReplacementFor(StringRef FunctionName,` 展开声明或语句。

### Lines 54-63
```cpp
  54 |       return AnnexKReplacementFunction;
  55 |   }
  56 | 
  57 |   // FIXME: Some of these functions are available in C++ under "std::", and
  58 |   // should be matched and suggested.
  59 |   return StringSwitch<StringRef>(FunctionName)
  60 |       .Cases({"asctime", "asctime_r"}, "strftime")
  61 |       .Case("gets", "fgets")
  62 |       .Case("rewind", "fseek")
  63 |       .Case("setbuf", "setvbuf")
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 64-73
```cpp
  64 |       .Case("get_temporary_buffer", "operator new[]");
  65 | }
  66 | 
  67 | static StringRef getReplacementForAdditional(StringRef FunctionName,
  68 |                                              bool IsAnnexKAvailable) {
  69 |   if (IsAnnexKAvailable) {
  70 |     // Try to find a better replacement from Annex K first.
  71 |     StringRef AnnexKReplacementFunction = StringSwitch<StringRef>(FunctionName)
  72 |                                               .Case("bcopy", "memcpy_s")
  73 |                                               .Case("bzero", "memset_s")
```
- EN: This block continues the implementation with declarations or statements centered on `.Case("get_temporary_buffer", "operator new[]");`.
- CN: 这一段继续实现，围绕 `.Case("get_temporary_buffer", "operator new[]");` 展开声明或语句。

### Lines 74-79
```cpp
  74 |                                               .Default({});
  75 | 
  76 |     if (!AnnexKReplacementFunction.empty())
  77 |       return AnnexKReplacementFunction;
  78 |   }
  79 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 80-87
```cpp
  80 |   return StringSwitch<StringRef>(FunctionName)
  81 |       .Case("bcmp", "memcmp")
  82 |       .Case("bcopy", "memcpy")
  83 |       .Case("bzero", "memset")
  84 |       .Case("getpw", "getpwuid")
  85 |       .Case("vfork", "posix_spawn");
  86 | }
  87 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 88-97
```cpp
  88 | /// \returns The rationale for replacing the function \p FunctionName with the
  89 | /// safer alternative.
  90 | static StringRef getRationaleFor(StringRef FunctionName) {
  91 |   return StringSwitch<StringRef>(FunctionName)
  92 |       .Cases({"asctime", "asctime_r", "ctime"},
  93 |              "is not bounds-checking and non-reentrant")
  94 |       .Cases({"bcmp", "bcopy", "bzero"}, "is deprecated")
  95 |       .Cases({"fopen", "freopen"}, "has no exclusive access to the opened file")
  96 |       .Case("gets", "is insecure, was deprecated and removed in C11 and C++14")
  97 |       .Case("getpw", "is dangerous as it may overflow the provided buffer")
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 98-106
```cpp
  98 |       .Cases({"rewind", "setbuf"}, "has no error detection")
  99 |       .Case("vfork", "is insecure as it can lead to denial of service "
 100 |                      "situations in the parent process")
 101 |       .Case("get_temporary_buffer", "returns uninitialized memory without "
 102 |                                     "performance advantages, was deprecated in "
 103 |                                     "C++17 and removed in C++20")
 104 |       .Default("is not bounds-checking");
 105 | }
 106 | 
```
- EN: This block continues the implementation with declarations or statements centered on `.Cases({"rewind", "setbuf"}, "has no error detection")`.
- CN: 这一段继续实现，围绕 `.Cases({"rewind", "setbuf"}, "has no error detection")` 展开声明或语句。

### Lines 107-115
```cpp
 107 | /// Calculates whether Annex K is available for the current translation unit
 108 | /// based on the macro definitions and the language options.
 109 | ///
 110 | /// The result is cached and saved in \p CacheVar.
 111 | static bool isAnnexKAvailable(std::optional<bool> &CacheVar, Preprocessor *PP,
 112 |                               const LangOptions &LO) {
 113 |   if (CacheVar.has_value())
 114 |     return *CacheVar;
 115 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 116-119
```cpp
 116 |   CacheVar = analysis::isAnnexKAvailable(PP, LO);
 117 |   return CacheVar.value();
 118 | }
 119 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 120-126
```cpp
 120 | static std::vector<UnsafeFunctionsCheck::CheckedFunction>
 121 | parseCheckedFunctions(StringRef Option, ClangTidyContext *Context) {
 122 |   const std::vector<StringRef> Functions =
 123 |       utils::options::parseStringList(Option);
 124 |   std::vector<UnsafeFunctionsCheck::CheckedFunction> Result;
 125 |   Result.reserve(Functions.size());
 126 | 
```
- EN: This block continues the implementation with declarations or statements centered on `static std::vector<UnsafeFunctionsCheck::CheckedFunction>`.
- CN: 这一段继续实现，围绕 `static std::vector<UnsafeFunctionsCheck::CheckedFunction>` 展开声明或语句。

### Lines 127-130
```cpp
 127 |   for (const StringRef Function : Functions) {
 128 |     if (Function.empty())
 129 |       continue;
 130 | 
```
- EN: This block continues the implementation with declarations or statements centered on `for (const StringRef Function : Functions) {`.
- CN: 这一段继续实现，围绕 `for (const StringRef Function : Functions) {` 展开声明或语句。

### Lines 131-140
```cpp
 131 |     const auto [Name, Rest] = Function.split(',');
 132 |     const auto [Replacement, Reason] = Rest.split(',');
 133 | 
 134 |     if (Name.trim().empty()) {
 135 |       Context->configurationDiag("invalid configuration value for option '%0'; "
 136 |                                  "expected the name of an unsafe function")
 137 |           << OptionNameCustomFunctions;
 138 |       continue;
 139 |     }
 140 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const auto [Name, Rest] = Function.split(',');`.
- CN: 这一段继续实现，围绕 `const auto [Name, Rest] = Function.split(',');` 展开声明或语句。

### Lines 141-146
```cpp
 141 |     Result.push_back(
 142 |         {Name.trim().str(),
 143 |          matchers::MatchesAnyListedRegexNameMatcher::NameMatcher(Name.trim()),
 144 |          Replacement.trim().str(), Reason.trim().str()});
 145 |   }
 146 | 
```
- EN: This block continues the implementation with declarations or statements centered on `Result.push_back(`.
- CN: 这一段继续实现，围绕 `Result.push_back(` 展开声明或语句。

### Lines 147-154
```cpp
 147 |   return Result;
 148 | }
 149 | 
 150 | static std::string serializeCheckedFunctions(
 151 |     const std::vector<UnsafeFunctionsCheck::CheckedFunction> &Functions) {
 152 |   std::vector<std::string> Result;
 153 |   Result.reserve(Functions.size());
 154 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 155-161
```cpp
 155 |   for (const auto &Entry : Functions)
 156 |     if (Entry.Reason.empty())
 157 |       Result.push_back(Entry.Name + "," + Entry.Replacement);
 158 |     else
 159 |       Result.push_back(Entry.Name + "," + Entry.Replacement + "," +
 160 |                        Entry.Reason);
 161 | 
```
- EN: This block continues the implementation with declarations or statements centered on `for (const auto &Entry : Functions)`.
- CN: 这一段继续实现，围绕 `for (const auto &Entry : Functions)` 展开声明或语句。

### Lines 162-171
```cpp
 162 |   return llvm::join(Result, ";");
 163 | }
 164 | 
 165 | UnsafeFunctionsCheck::UnsafeFunctionsCheck(StringRef Name,
 166 |                                            ClangTidyContext *Context)
 167 |     : ClangTidyCheck(Name, Context),
 168 |       CustomFunctions(parseCheckedFunctions(
 169 |           Options.get(OptionNameCustomFunctions, ""), Context)),
 170 |       ReportDefaultFunctions(
 171 |           Options.get(OptionNameReportDefaultFunctions, true)),
```
- EN: Method definitions such as `llvm::join`, `UnsafeFunctionsCheck::UnsafeFunctionsCheck` provide the concrete behavior declared elsewhere.
- CN: 诸如 `llvm::join`、`UnsafeFunctionsCheck::UnsafeFunctionsCheck` 的方法定义给出了前面声明的具体行为。
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 172-181
```cpp
 172 |       ReportMoreUnsafeFunctions(
 173 |           Options.get(OptionNameReportMoreUnsafeFunctions, true)) {}
 174 | 
 175 | void UnsafeFunctionsCheck::storeOptions(ClangTidyOptions::OptionMap &Opts) {
 176 |   Options.store(Opts, OptionNameCustomFunctions,
 177 |                 serializeCheckedFunctions(CustomFunctions));
 178 |   Options.store(Opts, OptionNameReportDefaultFunctions, ReportDefaultFunctions);
 179 |   Options.store(Opts, OptionNameReportMoreUnsafeFunctions,
 180 |                 ReportMoreUnsafeFunctions);
 181 | }
```
- EN: Method definitions such as `UnsafeFunctionsCheck::storeOptions` provide the concrete behavior declared elsewhere.
- CN: 诸如 `UnsafeFunctionsCheck::storeOptions` 的方法定义给出了前面声明的具体行为。

### Lines 182-191
```cpp
 182 | 
 183 | void UnsafeFunctionsCheck::registerMatchers(MatchFinder *Finder) {
 184 |   if (ReportDefaultFunctions) {
 185 |     if (getLangOpts().C11) {
 186 |       // Matching functions with safe replacements only in Annex K.
 187 |       auto FunctionNamesWithAnnexKReplacementMatcher = hasAnyName(
 188 |           "::bsearch", "::ctime", "::fopen", "::fprintf", "::freopen",
 189 |           "::fscanf", "::fwprintf", "::fwscanf", "::getenv", "::gmtime",
 190 |           "::localtime", "::mbsrtowcs", "::mbstowcs", "::memcpy", "::memmove",
 191 |           "::memset", "::printf", "::qsort", "::scanf", "::snprintf",
```
- EN: This block defines AST matcher registration, i.e. the structural patterns the check will look for.
- CN: 这一段定义 AST 匹配器注册，也就是该检查要查找的结构模式。
- EN: Method definitions such as `UnsafeFunctionsCheck::registerMatchers` provide the concrete behavior declared elsewhere.
- CN: 诸如 `UnsafeFunctionsCheck::registerMatchers` 的方法定义给出了前面声明的具体行为。

### Lines 192-201
```cpp
 192 |           "::sprintf", "::sscanf", "::strcat", "::strcpy", "::strerror",
 193 |           "::strlen", "::strncat", "::strncpy", "::strtok", "::swprintf",
 194 |           "::swscanf", "::vfprintf", "::vfscanf", "::vfwprintf", "::vfwscanf",
 195 |           "::vprintf", "::vscanf", "::vsnprintf", "::vsprintf", "::vsscanf",
 196 |           "::vswprintf", "::vswscanf", "::vwprintf", "::vwscanf", "::wcrtomb",
 197 |           "::wcscat", "::wcscpy", "::wcslen", "::wcsncat", "::wcsncpy",
 198 |           "::wcsrtombs", "::wcstok", "::wcstombs", "::wctomb", "::wmemcpy",
 199 |           "::wmemmove", "::wprintf", "::wscanf");
 200 |       Finder->addMatcher(
 201 |           declRefExpr(to(functionDecl(FunctionNamesWithAnnexKReplacementMatcher)
```
- EN: This block continues the implementation with declarations or statements centered on `"::sprintf", "::sscanf", "::strcat", "::strcpy", "::strerror`.
- CN: 这一段继续实现，围绕 `"::sprintf", "::sscanf", "::strcat", "::strcpy", "::strerror` 展开声明或语句。

### Lines 202-206
```cpp
 202 |                              .bind(FunctionNamesWithAnnexKReplacementId)))
 203 |               .bind(DeclRefId),
 204 |           this);
 205 |     }
 206 | 
```
- EN: This block continues the implementation with declarations or statements centered on `.bind(FunctionNamesWithAnnexKReplacementId)))`.
- CN: 这一段继续实现，围绕 `.bind(FunctionNamesWithAnnexKReplacementId)))` 展开声明或语句。

### Lines 207-216
```cpp
 207 |     // Matching functions with replacements without Annex K.
 208 |     auto FunctionNamesMatcher =
 209 |         hasAnyName("::asctime", "asctime_r", "::gets", "::rewind", "::setbuf",
 210 |                    "::std::get_temporary_buffer");
 211 |     Finder->addMatcher(
 212 |         declRefExpr(
 213 |             to(functionDecl(FunctionNamesMatcher).bind(FunctionNamesId)))
 214 |             .bind(DeclRefId),
 215 |         this);
 216 | 
```
- EN: This block continues the implementation with declarations or statements centered on `// Matching functions with replacements without Annex K.`.
- CN: 这一段继续实现，围绕 `// Matching functions with replacements without Annex K.` 展开声明或语句。

### Lines 217-226
```cpp
 217 |     if (ReportMoreUnsafeFunctions) {
 218 |       // Matching functions with replacements without Annex K, at user request.
 219 |       auto AdditionalFunctionNamesMatcher =
 220 |           hasAnyName("::bcmp", "::bcopy", "::bzero", "::getpw", "::vfork");
 221 |       Finder->addMatcher(
 222 |           declRefExpr(to(functionDecl(AdditionalFunctionNamesMatcher)
 223 |                              .bind(AdditionalFunctionNamesId)))
 224 |               .bind(DeclRefId),
 225 |           this);
 226 |     }
```
- EN: This block continues the implementation with declarations or statements centered on `if (ReportMoreUnsafeFunctions) {`.
- CN: 这一段继续实现，围绕 `if (ReportMoreUnsafeFunctions) {` 展开声明或语句。

### Lines 227-232
```cpp
 227 |   }
 228 | 
 229 |   if (!CustomFunctions.empty()) {
 230 |     std::vector<StringRef> FunctionNames;
 231 |     FunctionNames.reserve(CustomFunctions.size());
 232 | 
```
- EN: This block continues the implementation with declarations or statements centered on `}`.
- CN: 这一段继续实现，围绕 `}` 展开声明或语句。

### Lines 233-238
```cpp
 233 |     for (const auto &Entry : CustomFunctions)
 234 |       FunctionNames.emplace_back(Entry.Name);
 235 | 
 236 |     auto CustomFunctionsMatcher =
 237 |         matchers::matchesAnyListedRegexName(FunctionNames);
 238 | 
```
- EN: Method definitions such as `matchers::matchesAnyListedRegexName` provide the concrete behavior declared elsewhere.
- CN: 诸如 `matchers::matchesAnyListedRegexName` 的方法定义给出了前面声明的具体行为。

### Lines 239-248
```cpp
 239 |     Finder->addMatcher(declRefExpr(to(functionDecl(CustomFunctionsMatcher)
 240 |                                           .bind(CustomFunctionNamesId)))
 241 |                            .bind(DeclRefId),
 242 |                        this);
 243 |     // C++ member calls do not contain a DeclRefExpr to the function decl.
 244 |     // Instead, they contain a MemberExpr that refers to the decl.
 245 |     Finder->addMatcher(memberExpr(member(functionDecl(CustomFunctionsMatcher)
 246 |                                              .bind(CustomFunctionNamesId)))
 247 |                            .bind(DeclRefId),
 248 |                        this);
```
- EN: This block continues the implementation with declarations or statements centered on `Finder->addMatcher(declRefExpr(to(functionDecl(CustomFunctio`.
- CN: 这一段继续实现，围绕 `Finder->addMatcher(declRefExpr(to(functionDecl(CustomFunctio` 展开声明或语句。

### Lines 249-258
```cpp
 249 |   }
 250 | }
 251 | 
 252 | /// A ``Reason`` prefixed with ``>`` produces a fully-custom message and
 253 | /// suppresses the ``Replacement`` suffix; an empty ``Replacement`` yields
 254 | /// the "it should not be used" form; otherwise the standard suggestion
 255 | /// form is used.
 256 | static void emitDiag(ClangTidyCheck &Check, const Expr *SourceExpr,
 257 |                      const FunctionDecl *FuncDecl, StringRef Replacement,
 258 |                      StringRef Reason) {
```
- EN: This block continues the implementation with declarations or statements centered on `}`.
- CN: 这一段继续实现，围绕 `}` 展开声明或语句。

### Lines 259-268
```cpp
 259 |   if (Reason.consume_front(">")) {
 260 |     Check.diag(SourceExpr->getExprLoc(), "function %0 %1")
 261 |         << FuncDecl << Reason.trim() << SourceExpr->getSourceRange();
 262 |     return;
 263 |   }
 264 |   if (Replacement.empty()) {
 265 |     Check.diag(SourceExpr->getExprLoc(),
 266 |                "function %0 %1; it should not be used")
 267 |         << FuncDecl << Reason << SourceExpr->getSourceRange();
 268 |     return;
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

### Lines 269-274
```cpp
 269 |   }
 270 |   Check.diag(SourceExpr->getExprLoc(),
 271 |              "function %0 %1; '%2' should be used instead")
 272 |       << FuncDecl << Reason << Replacement << SourceExpr->getSourceRange();
 273 | }
 274 | 
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

### Lines 275-278
```cpp
 275 | void UnsafeFunctionsCheck::check(const MatchFinder::MatchResult &Result) {
 276 |   const Expr *SourceExpr = nullptr;
 277 |   const FunctionDecl *FuncDecl = nullptr;
 278 | 
```
- EN: Method definitions such as `UnsafeFunctionsCheck::check` provide the concrete behavior declared elsewhere.
- CN: 诸如 `UnsafeFunctionsCheck::check` 的方法定义给出了前面声明的具体行为。

### Lines 279-288
```cpp
 279 |   if (const auto *DeclRef = Result.Nodes.getNodeAs<DeclRefExpr>(DeclRefId)) {
 280 |     SourceExpr = DeclRef;
 281 |     FuncDecl = cast<FunctionDecl>(DeclRef->getDecl());
 282 |   } else if (const auto *Member =
 283 |                  Result.Nodes.getNodeAs<MemberExpr>(DeclRefId)) {
 284 |     SourceExpr = Member;
 285 |     FuncDecl = cast<FunctionDecl>(Member->getMemberDecl());
 286 |   } else {
 287 |     llvm_unreachable("No valid matched node in check()");
 288 |     return;
```
- EN: This block continues the implementation with declarations or statements centered on `if (const auto *DeclRef = Result.Nodes.getNodeAs<DeclRefExpr`.
- CN: 这一段继续实现，围绕 `if (const auto *DeclRef = Result.Nodes.getNodeAs<DeclRefExpr` 展开声明或语句。

### Lines 289-292
```cpp
 289 |   }
 290 | 
 291 |   assert(SourceExpr && FuncDecl && "No valid matched node in check()");
 292 | 
```
- EN: This block continues the implementation with declarations or statements centered on `}`.
- CN: 这一段继续实现，围绕 `}` 展开声明或语句。

### Lines 293-302
```cpp
 293 |   // Only one of these are matched at a time.
 294 |   const auto *AnnexK = Result.Nodes.getNodeAs<FunctionDecl>(
 295 |       FunctionNamesWithAnnexKReplacementId);
 296 |   const auto *Normal = Result.Nodes.getNodeAs<FunctionDecl>(FunctionNamesId);
 297 |   const auto *Additional =
 298 |       Result.Nodes.getNodeAs<FunctionDecl>(AdditionalFunctionNamesId);
 299 |   const auto *Custom =
 300 |       Result.Nodes.getNodeAs<FunctionDecl>(CustomFunctionNamesId);
 301 |   assert((AnnexK || Normal || Additional || Custom) &&
 302 |          "No valid match category.");
```
- EN: This block continues the implementation with declarations or statements centered on `// Only one of these are matched at a time.`.
- CN: 这一段继续实现，围绕 `// Only one of these are matched at a time.` 展开声明或语句。

### Lines 303-307
```cpp
 303 | 
 304 |   bool AnnexKIsAvailable =
 305 |       isAnnexKAvailable(IsAnnexKAvailable, PP, getLangOpts());
 306 |   StringRef FunctionName = FuncDecl->getName();
 307 | 
```
- EN: This block continues the implementation with declarations or statements centered on `bool AnnexKIsAvailable =`.
- CN: 这一段继续实现，围绕 `bool AnnexKIsAvailable =` 展开声明或语句。

### Lines 308-317
```cpp
 308 |   std::string Replacement;
 309 |   std::string Reason;
 310 | 
 311 |   if (Custom) {
 312 |     const CheckedFunction *MatchedEntry = nullptr;
 313 |     for (const auto &Entry : CustomFunctions) {
 314 |       if (Entry.Pattern.match(*FuncDecl)) {
 315 |         MatchedEntry = &Entry;
 316 |         break;
 317 |       }
```
- EN: This block continues the implementation with declarations or statements centered on `std::string Replacement;`.
- CN: 这一段继续实现，围绕 `std::string Replacement;` 展开声明或语句。

### Lines 318-327
```cpp
 318 |     }
 319 |     if (!MatchedEntry) {
 320 |       llvm_unreachable("No custom function was matched.");
 321 |       return;
 322 |     }
 323 |     Replacement = MatchedEntry->Replacement;
 324 |     Reason = MatchedEntry->Reason.empty() ? "is marked as unsafe"
 325 |                                           : MatchedEntry->Reason;
 326 |   } else {
 327 |     const std::optional<std::string> ReplacementFunctionName =
```
- EN: This block continues the implementation with declarations or statements centered on `}`.
- CN: 这一段继续实现，围绕 `}` 展开声明或语句。

### Lines 328-334
```cpp
 328 |         [&]() -> std::optional<std::string> {
 329 |       if (AnnexK) {
 330 |         if (AnnexKIsAvailable)
 331 |           return getAnnexKReplacementFor(FunctionName);
 332 |         return std::nullopt;
 333 |       }
 334 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 335-341
```cpp
 335 |       if (Normal)
 336 |         return getReplacementFor(FunctionName, AnnexKIsAvailable).str();
 337 | 
 338 |       if (Additional)
 339 |         return getReplacementForAdditional(FunctionName, AnnexKIsAvailable)
 340 |             .str();
 341 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 342-346
```cpp
 342 |       llvm_unreachable("Unhandled match category");
 343 |     }();
 344 |     if (!ReplacementFunctionName)
 345 |       return;
 346 | 
```
- EN: This block continues the implementation with declarations or statements centered on `llvm_unreachable("Unhandled match category");`.
- CN: 这一段继续实现，围绕 `llvm_unreachable("Unhandled match category");` 展开声明或语句。

### Lines 347-350
```cpp
 347 |     Replacement = *ReplacementFunctionName;
 348 |     Reason = getRationaleFor(FunctionName).str();
 349 |   }
 350 | 
```
- EN: This block continues the implementation with declarations or statements centered on `Replacement = *ReplacementFunctionName;`.
- CN: 这一段继续实现，围绕 `Replacement = *ReplacementFunctionName;` 展开声明或语句。

### Lines 351-359
```cpp
 351 |   emitDiag(*this, SourceExpr, FuncDecl, Replacement, Reason);
 352 | }
 353 | 
 354 | void UnsafeFunctionsCheck::registerPPCallbacks(
 355 |     const SourceManager &SM, Preprocessor *PP,
 356 |     Preprocessor * /*ModuleExpanderPP*/) {
 357 |   this->PP = PP;
 358 | }
 359 | 
```
- EN: Method definitions such as `UnsafeFunctionsCheck::registerPPCallbacks` provide the concrete behavior declared elsewhere.
- CN: 诸如 `UnsafeFunctionsCheck::registerPPCallbacks` 的方法定义给出了前面声明的具体行为。

### Lines 360-365
```cpp
 360 | void UnsafeFunctionsCheck::onEndOfTranslationUnit() {
 361 |   this->PP = nullptr;
 362 |   IsAnnexKAvailable.reset();
 363 | }
 364 | 
 365 | } // namespace clang::tidy::bugprone
```
- EN: Method definitions such as `UnsafeFunctionsCheck::onEndOfTranslationUnit` provide the concrete behavior declared elsewhere.
- CN: 诸如 `UnsafeFunctionsCheck::onEndOfTranslationUnit` 的方法定义给出了前面声明的具体行为。

## Key Concepts / 关键概念
- clang-tidy check lifecycle / clang-tidy 检查生命周期
- AST matcher DSL / AST 匹配器 DSL
- diagnostics and fix-its / 诊断与修复建议
- namespace scoping / 命名空间作用域

## Dependencies / 依赖关系
- EN: Direct includes: `UnsafeFunctionsCheck.h`, `../utils/OptionsUtils.h`, `clang/AST/ASTContext.h`, `clang/ASTMatchers/ASTMatchFinder.h`, `clang/Analysis/AnnexKDetection.h`, `clang/Lex/Preprocessor.h`, `cassert`.
- CN: 直接包含依赖: `UnsafeFunctionsCheck.h`、`../utils/OptionsUtils.h`、`clang/AST/ASTContext.h`、`clang/ASTMatchers/ASTMatchFinder.h`、`clang/Analysis/AnnexKDetection.h`、`clang/Lex/Preprocessor.h`、`cassert`。
- EN: Namespace context: `clang::tidy::bugprone`.
- CN: 命名空间上下文: `clang::tidy::bugprone`。
