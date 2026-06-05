# StringFindStrContainsCheck.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `clang-tools-extra/clang-tidy/abseil/StringFindStrContainsCheck.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `StringFindStrContainsCheck` with clang transformer rewrite rules and fix-it generation.
- 用途 (CN): 使用 clang transformer 重写规则与修复建议实现 `StringFindStrContainsCheck`。

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
   9 | #include "StringFindStrContainsCheck.h"
  10 | 
  11 | #include "../utils/OptionsUtils.h"
  12 | #include "clang/AST/ASTContext.h"
  13 | #include "clang/ASTMatchers/ASTMatchers.h"
  14 | #include "clang/Tooling/Transformer/RewriteRule.h"
  15 | #include "clang/Tooling/Transformer/Stencil.h"
  16 | 
```
- EN: The section imports dependencies such as `StringFindStrContainsCheck.h`, `../utils/OptionsUtils.h`, `clang/AST/ASTContext.h`, `clang/ASTMatchers/ASTMatchers.h` needed by this file.
- CN: 本段引入了 `StringFindStrContainsCheck.h`、`../utils/OptionsUtils.h`、`clang/AST/ASTContext.h`、`clang/ASTMatchers/ASTMatchers.h` 等依赖，供当前文件使用。
- EN: It builds a transformer rewrite rule that matches source patterns and describes the replacement text.
- CN: 这里构建了 transformer 重写规则：匹配源码模式并描述替换文本。

### Lines 17-21
```cpp
  17 | // FixItHint - Hint to check documentation script to mark this check as
  18 | // providing a FixIt.
  19 | 
  20 | using namespace clang::ast_matchers;
  21 | 
```
- EN: Using-directives shorten qualified names so matcher/transformer code reads more directly.
- CN: using 指令缩短了限定名，使匹配器/变换器代码更直接。

### Lines 22-31
```cpp
  22 | namespace clang::tidy::abseil {
  23 | 
  24 | using ::clang::transformer::addInclude;
  25 | using ::clang::transformer::applyFirst;
  26 | using ::clang::transformer::cat;
  27 | using ::clang::transformer::changeTo;
  28 | using ::clang::transformer::makeRule;
  29 | using ::clang::transformer::node;
  30 | using ::clang::transformer::RewriteRuleWith;
  31 | 
```
- EN: Namespace scopes such as `clang::tidy::abseil` place the symbols in their intended subsystem.
- CN: 诸如 `clang::tidy::abseil` 这样的命名空间将符号放入预期的子系统中。
- EN: It builds a transformer rewrite rule that matches source patterns and describes the replacement text.
- CN: 这里构建了 transformer 重写规则：匹配源码模式并描述替换文本。

### Lines 32-35
```cpp
  32 | namespace {
  33 | AST_MATCHER(Type, isCharType) { return Node.isCharType(); }
  34 | } // namespace
  35 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 36-41
```cpp
  36 | static constexpr char DefaultStringLikeClasses[] = "::std::basic_string;"
  37 |                                                    "::std::basic_string_view;"
  38 |                                                    "::absl::string_view";
  39 | static constexpr char DefaultAbseilStringsMatchHeader[] =
  40 |     "absl/strings/match.h";
  41 | 
```
- EN: This block continues the implementation with declarations or statements centered on `static constexpr char DefaultStringLikeClasses[] = "::std::b`.
- CN: 这一段继续实现，围绕 `static constexpr char DefaultStringLikeClasses[] = "::std::b` 展开声明或语句。

### Lines 42-51
```cpp
  42 | static transformer::RewriteRuleWith<std::string>
  43 | makeRewriteRule(ArrayRef<StringRef> StringLikeClassNames,
  44 |                 StringRef AbseilStringsMatchHeader) {
  45 |   auto StringLikeClass = cxxRecordDecl(hasAnyName(StringLikeClassNames));
  46 |   auto StringType =
  47 |       hasUnqualifiedDesugaredType(recordType(hasDeclaration(StringLikeClass)));
  48 |   auto CharStarType =
  49 |       hasUnqualifiedDesugaredType(pointerType(pointee(isAnyCharacter())));
  50 |   auto CharType = hasUnqualifiedDesugaredType(isCharType());
  51 |   auto StringNpos = declRefExpr(
```
- EN: It builds a transformer rewrite rule that matches source patterns and describes the replacement text.
- CN: 这里构建了 transformer 重写规则：匹配源码模式并描述替换文本。

### Lines 52-61
```cpp
  52 |       to(varDecl(hasName("npos"), hasDeclContext(StringLikeClass))));
  53 |   auto StringFind = cxxMemberCallExpr(
  54 |       callee(cxxMethodDecl(
  55 |           hasName("find"), parameterCountIs(2),
  56 |           hasParameter(
  57 |               0, parmVarDecl(anyOf(hasType(StringType), hasType(CharStarType),
  58 |                                    hasType(CharType)))))),
  59 |       on(hasType(StringType)), hasArgument(0, expr().bind("parameter_to_find")),
  60 |       anyOf(hasArgument(1, integerLiteral(equals(0))),
  61 |             hasArgument(1, cxxDefaultArgExpr())),
```
- EN: This block continues the implementation with declarations or statements centered on `to(varDecl(hasName("npos"), hasDeclContext(StringLikeClass))`.
- CN: 这一段继续实现，围绕 `to(varDecl(hasName("npos"), hasDeclContext(StringLikeClass))` 展开声明或语句。

### Lines 62-71
```cpp
  62 |       onImplicitObjectArgument(expr().bind("string_being_searched")));
  63 | 
  64 |   RewriteRuleWith<std::string> Rule = applyFirst(
  65 |       {makeRule(
  66 |            binaryOperator(hasOperatorName("=="),
  67 |                           hasOperands(ignoringParenImpCasts(StringNpos),
  68 |                                       ignoringParenImpCasts(StringFind))),
  69 |            {changeTo(cat("!absl::StrContains(", node("string_being_searched"),
  70 |                          ", ", node("parameter_to_find"), ")")),
  71 |             addInclude(AbseilStringsMatchHeader)},
```
- EN: It builds a transformer rewrite rule that matches source patterns and describes the replacement text.
- CN: 这里构建了 transformer 重写规则：匹配源码模式并描述替换文本。

### Lines 72-81
```cpp
  72 |            cat("use !absl::StrContains instead of find() == npos")),
  73 |        makeRule(
  74 |            binaryOperator(hasOperatorName("!="),
  75 |                           hasOperands(ignoringParenImpCasts(StringNpos),
  76 |                                       ignoringParenImpCasts(StringFind))),
  77 |            {changeTo(cat("absl::StrContains(", node("string_being_searched"),
  78 |                          ", ", node("parameter_to_find"), ")")),
  79 |             addInclude(AbseilStringsMatchHeader)},
  80 |            cat("use absl::StrContains instead "
  81 |                "of find() != npos"))});
```
- EN: It builds a transformer rewrite rule that matches source patterns and describes the replacement text.
- CN: 这里构建了 transformer 重写规则：匹配源码模式并描述替换文本。

### Lines 82-91
```cpp
  82 |   return Rule;
  83 | }
  84 | 
  85 | StringFindStrContainsCheck::StringFindStrContainsCheck(
  86 |     StringRef Name, ClangTidyContext *Context)
  87 |     : TransformerClangTidyCheck(Name, Context),
  88 |       StringLikeClassesOption(utils::options::parseStringList(
  89 |           Options.get("StringLikeClasses", DefaultStringLikeClasses))),
  90 |       AbseilStringsMatchHeaderOption(Options.get(
  91 |           "AbseilStringsMatchHeader", DefaultAbseilStringsMatchHeader)) {
```
- EN: Method definitions such as `StringFindStrContainsCheck::StringFindStrContainsCheck` provide the concrete behavior declared elsewhere.
- CN: 诸如 `StringFindStrContainsCheck::StringFindStrContainsCheck` 的方法定义给出了前面声明的具体行为。
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 92-95
```cpp
  92 |   setRule(
  93 |       makeRewriteRule(StringLikeClassesOption, AbseilStringsMatchHeaderOption));
  94 | }
  95 | 
```
- EN: It builds a transformer rewrite rule that matches source patterns and describes the replacement text.
- CN: 这里构建了 transformer 重写规则：匹配源码模式并描述替换文本。

### Lines 96-100
```cpp
  96 | bool StringFindStrContainsCheck::isLanguageVersionSupported(
  97 |     const LangOptions &LangOpts) const {
  98 |   return LangOpts.CPlusPlus11;
  99 | }
 100 | 
```
- EN: This logic gates the check on a specific language mode so it only runs when the feature is available.
- CN: 这里按特定语言模式进行门控，确保检查只在相应特性可用时运行。
- EN: Method definitions such as `StringFindStrContainsCheck::isLanguageVersionSupported` provide the concrete behavior declared elsewhere.
- CN: 诸如 `StringFindStrContainsCheck::isLanguageVersionSupported` 的方法定义给出了前面声明的具体行为。
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 101-110
```cpp
 101 | void StringFindStrContainsCheck::storeOptions(
 102 |     ClangTidyOptions::OptionMap &Opts) {
 103 |   TransformerClangTidyCheck::storeOptions(Opts);
 104 |   Options.store(Opts, "StringLikeClasses",
 105 |                 utils::options::serializeStringList(StringLikeClassesOption));
 106 |   Options.store(Opts, "AbseilStringsMatchHeader",
 107 |                 AbseilStringsMatchHeaderOption);
 108 | }
 109 | 
 110 | } // namespace clang::tidy::abseil
```
- EN: Method definitions such as `StringFindStrContainsCheck::storeOptions`, `TransformerClangTidyCheck::storeOptions` provide the concrete behavior declared elsewhere.
- CN: 诸如 `StringFindStrContainsCheck::storeOptions`、`TransformerClangTidyCheck::storeOptions` 的方法定义给出了前面声明的具体行为。

## Key Concepts / 关键概念
- clang-tidy check lifecycle / clang-tidy 检查生命周期
- AST matcher DSL / AST 匹配器 DSL
- source-to-source rewriting / 源码到源码的重写
- language-version gating / 语言版本门控
- namespace scoping / 命名空间作用域

## Dependencies / 依赖关系
- EN: Direct includes: `StringFindStrContainsCheck.h`, `../utils/OptionsUtils.h`, `clang/AST/ASTContext.h`, `clang/ASTMatchers/ASTMatchers.h`, `clang/Tooling/Transformer/RewriteRule.h`, `clang/Tooling/Transformer/Stencil.h`.
- CN: 直接包含依赖: `StringFindStrContainsCheck.h`、`../utils/OptionsUtils.h`、`clang/AST/ASTContext.h`、`clang/ASTMatchers/ASTMatchers.h`、`clang/Tooling/Transformer/RewriteRule.h`、`clang/Tooling/Transformer/Stencil.h`。
- EN: Namespace context: `clang::tidy::abseil`.
- CN: 命名空间上下文: `clang::tidy::abseil`。
