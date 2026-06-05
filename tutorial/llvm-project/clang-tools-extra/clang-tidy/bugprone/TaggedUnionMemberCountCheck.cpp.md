# TaggedUnionMemberCountCheck.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `clang-tools-extra/clang-tidy/bugprone/TaggedUnionMemberCountCheck.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `TaggedUnionMemberCountCheck` by registering AST matchers and producing diagnostics/fix-its.
- 用途 (CN): 通过注册 AST 匹配器并生成诊断/修复建议来实现 `TaggedUnionMemberCountCheck`。

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
   9 | #include "TaggedUnionMemberCountCheck.h"
  10 | #include "../utils/OptionsUtils.h"
  11 | #include "clang/ASTMatchers/ASTMatchFinder.h"
  12 | #include "llvm/ADT/STLExtras.h"
  13 | #include "llvm/ADT/SmallSet.h"
  14 | 
```
- EN: The section imports dependencies such as `TaggedUnionMemberCountCheck.h`, `../utils/OptionsUtils.h`, `clang/ASTMatchers/ASTMatchFinder.h`, `llvm/ADT/STLExtras.h` needed by this file.
- CN: 本段引入了 `TaggedUnionMemberCountCheck.h`、`../utils/OptionsUtils.h`、`clang/ASTMatchers/ASTMatchFinder.h`、`llvm/ADT/STLExtras.h` 等依赖，供当前文件使用。

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

### Lines 19-26
```cpp
  19 | static constexpr StringRef StrictModeOptionName = "StrictMode";
  20 | static constexpr StringRef EnableCountingEnumHeuristicOptionName =
  21 |     "EnableCountingEnumHeuristic";
  22 | static constexpr StringRef CountingEnumPrefixesOptionName =
  23 |     "CountingEnumPrefixes";
  24 | static constexpr StringRef CountingEnumSuffixesOptionName =
  25 |     "CountingEnumSuffixes";
  26 | 
```
- EN: This block continues the implementation with declarations or statements centered on `static constexpr StringRef StrictModeOptionName = "StrictMod`.
- CN: 这一段继续实现，围绕 `static constexpr StringRef StrictModeOptionName = "StrictMod` 展开声明或语句。

### Lines 27-31
```cpp
  27 | static constexpr bool StrictModeOptionDefaultValue = false;
  28 | static constexpr bool EnableCountingEnumHeuristicOptionDefaultValue = true;
  29 | static constexpr StringRef CountingEnumPrefixesOptionDefaultValue = "";
  30 | static constexpr StringRef CountingEnumSuffixesOptionDefaultValue = "count";
  31 | 
```
- EN: This block continues the implementation with declarations or statements centered on `static constexpr bool StrictModeOptionDefaultValue = false;`.
- CN: 这一段继续实现，围绕 `static constexpr bool StrictModeOptionDefaultValue = false;` 展开声明或语句。

### Lines 32-35
```cpp
  32 | static constexpr StringRef RootMatchBindName = "root";
  33 | static constexpr StringRef UnionMatchBindName = "union";
  34 | static constexpr StringRef TagMatchBindName = "tags";
  35 | 
```
- EN: This block continues the implementation with declarations or statements centered on `static constexpr StringRef RootMatchBindName = "root";`.
- CN: 这一段继续实现，围绕 `static constexpr StringRef RootMatchBindName = "root";` 展开声明或语句。

### Lines 36-45
```cpp
  36 | namespace {
  37 | 
  38 | AST_MATCHER_P2(RecordDecl, fieldCountOfKindIsOne,
  39 |                ast_matchers::internal::Matcher<FieldDecl>, InnerMatcher,
  40 |                StringRef, BindName) {
  41 |   // BoundNodesTreeBuilder resets itself when a match occurs.
  42 |   // So to avoid losing previously saved binds, a temporary instance
  43 |   // is used for matching.
  44 |   //
  45 |   // For precedence, see commit: 5b07de1a5faf4a22ae6fd982b877c5e7e3a76559
```
- EN: This block continues the implementation with declarations or statements centered on `namespace {`.
- CN: 这一段继续实现，围绕 `namespace {` 展开声明或语句。

### Lines 46-55
```cpp
  46 |   ast_matchers::internal::BoundNodesTreeBuilder TempBuilder;
  47 | 
  48 |   const FieldDecl *FirstMatch = nullptr;
  49 |   for (const FieldDecl *Field : Node.fields()) {
  50 |     if (InnerMatcher.matches(*Field, Finder, &TempBuilder)) {
  51 |       if (FirstMatch)
  52 |         return false;
  53 |       FirstMatch = Field;
  54 |     }
  55 |   }
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 56-63
```cpp
  56 | 
  57 |   if (FirstMatch) {
  58 |     Builder->setBinding(BindName, DynTypedNode::create(*FirstMatch));
  59 |     return true;
  60 |   }
  61 |   return false;
  62 | }
  63 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 64-73
```cpp
  64 | } // namespace
  65 | 
  66 | TaggedUnionMemberCountCheck::TaggedUnionMemberCountCheck(
  67 |     StringRef Name, ClangTidyContext *Context)
  68 |     : ClangTidyCheck(Name, Context),
  69 |       StrictMode(
  70 |           Options.get(StrictModeOptionName, StrictModeOptionDefaultValue)),
  71 |       EnableCountingEnumHeuristic(
  72 |           Options.get(EnableCountingEnumHeuristicOptionName,
  73 |                       EnableCountingEnumHeuristicOptionDefaultValue)),
```
- EN: Method definitions such as `TaggedUnionMemberCountCheck::TaggedUnionMemberCountCheck` provide the concrete behavior declared elsewhere.
- CN: 诸如 `TaggedUnionMemberCountCheck::TaggedUnionMemberCountCheck` 的方法定义给出了前面声明的具体行为。

### Lines 74-83
```cpp
  74 |       CountingEnumPrefixes(utils::options::parseStringList(
  75 |           Options.get(CountingEnumPrefixesOptionName,
  76 |                       CountingEnumPrefixesOptionDefaultValue))),
  77 |       CountingEnumSuffixes(utils::options::parseStringList(
  78 |           Options.get(CountingEnumSuffixesOptionName,
  79 |                       CountingEnumSuffixesOptionDefaultValue))) {
  80 |   if (!EnableCountingEnumHeuristic) {
  81 |     if (Options.get(CountingEnumPrefixesOptionName))
  82 |       configurationDiag("%0: Counting enum heuristic is disabled but "
  83 |                         "%1 is set")
```
- EN: This block continues the implementation with declarations or statements centered on `CountingEnumPrefixes(utils::options::parseStringList(`.
- CN: 这一段继续实现，围绕 `CountingEnumPrefixes(utils::options::parseStringList(` 展开声明或语句。

### Lines 84-91
```cpp
  84 |           << Name << CountingEnumPrefixesOptionName;
  85 |     if (Options.get(CountingEnumSuffixesOptionName))
  86 |       configurationDiag("%0: Counting enum heuristic is disabled but "
  87 |                         "%1 is set")
  88 |           << Name << CountingEnumSuffixesOptionName;
  89 |   }
  90 | }
  91 | 
```
- EN: This block continues the implementation with declarations or statements centered on `<< Name << CountingEnumPrefixesOptionName;`.
- CN: 这一段继续实现，围绕 `<< Name << CountingEnumPrefixesOptionName;` 展开声明或语句。

### Lines 92-101
```cpp
  92 | void TaggedUnionMemberCountCheck::storeOptions(
  93 |     ClangTidyOptions::OptionMap &Opts) {
  94 |   Options.store(Opts, StrictModeOptionName, StrictMode);
  95 |   Options.store(Opts, EnableCountingEnumHeuristicOptionName,
  96 |                 EnableCountingEnumHeuristic);
  97 |   Options.store(Opts, CountingEnumPrefixesOptionName,
  98 |                 utils::options::serializeStringList(CountingEnumPrefixes));
  99 |   Options.store(Opts, CountingEnumSuffixesOptionName,
 100 |                 utils::options::serializeStringList(CountingEnumSuffixes));
 101 | }
```
- EN: Method definitions such as `TaggedUnionMemberCountCheck::storeOptions` provide the concrete behavior declared elsewhere.
- CN: 诸如 `TaggedUnionMemberCountCheck::storeOptions` 的方法定义给出了前面声明的具体行为。

### Lines 102-106
```cpp
 102 | 
 103 | void TaggedUnionMemberCountCheck::registerMatchers(MatchFinder *Finder) {
 104 |   auto NotFromSystemHeaderOrStdNamespace =
 105 |       unless(anyOf(isExpansionInSystemHeader(), isInStdNamespace()));
 106 | 
```
- EN: This block defines AST matcher registration, i.e. the structural patterns the check will look for.
- CN: 这一段定义 AST 匹配器注册，也就是该检查要查找的结构模式。
- EN: Method definitions such as `TaggedUnionMemberCountCheck::registerMatchers` provide the concrete behavior declared elsewhere.
- CN: 诸如 `TaggedUnionMemberCountCheck::registerMatchers` 的方法定义给出了前面声明的具体行为。

### Lines 107-110
```cpp
 107 |   auto UnionField =
 108 |       fieldDecl(hasType(qualType(hasCanonicalType(recordType(hasDeclaration(
 109 |           recordDecl(isUnion(), NotFromSystemHeaderOrStdNamespace)))))));
 110 | 
```
- EN: This block continues the implementation with declarations or statements centered on `auto UnionField =`.
- CN: 这一段继续实现，围绕 `auto UnionField =` 展开声明或语句。

### Lines 111-116
```cpp
 111 |   auto EnumField = fieldDecl(hasType(qualType(hasCanonicalType(
 112 |       enumType(hasDeclaration(enumDecl(NotFromSystemHeaderOrStdNamespace)))))));
 113 | 
 114 |   auto HasOneUnionField = fieldCountOfKindIsOne(UnionField, UnionMatchBindName);
 115 |   auto HasOneEnumField = fieldCountOfKindIsOne(EnumField, TagMatchBindName);
 116 | 
```
- EN: This block continues the implementation with declarations or statements centered on `auto EnumField = fieldDecl(hasType(qualType(hasCanonicalType`.
- CN: 这一段继续实现，围绕 `auto EnumField = fieldDecl(hasType(qualType(hasCanonicalType` 展开声明或语句。

### Lines 117-122
```cpp
 117 |   Finder->addMatcher(recordDecl(anyOf(isStruct(), isClass()), HasOneUnionField,
 118 |                                 HasOneEnumField, unless(isImplicit()))
 119 |                          .bind(RootMatchBindName),
 120 |                      this);
 121 | }
 122 | 
```
- EN: This block continues the implementation with declarations or statements centered on `Finder->addMatcher(recordDecl(anyOf(isStruct(), isClass()), `.
- CN: 这一段继续实现，围绕 `Finder->addMatcher(recordDecl(anyOf(isStruct(), isClass()), ` 展开声明或语句。

### Lines 123-132
```cpp
 123 | bool TaggedUnionMemberCountCheck::isCountingEnumLikeName(StringRef Name) const {
 124 |   if (llvm::any_of(CountingEnumPrefixes, [Name](StringRef Prefix) -> bool {
 125 |         return Name.starts_with_insensitive(Prefix);
 126 |       }))
 127 |     return true;
 128 |   if (llvm::any_of(CountingEnumSuffixes, [Name](StringRef Suffix) -> bool {
 129 |         return Name.ends_with_insensitive(Suffix);
 130 |       }))
 131 |     return true;
 132 |   return false;
```
- EN: Method definitions such as `TaggedUnionMemberCountCheck::isCountingEnumLikeName` provide the concrete behavior declared elsewhere.
- CN: 诸如 `TaggedUnionMemberCountCheck::isCountingEnumLikeName` 的方法定义给出了前面声明的具体行为。
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 133-138
```cpp
 133 | }
 134 | 
 135 | std::pair<const std::size_t, const EnumConstantDecl *>
 136 | TaggedUnionMemberCountCheck::getNumberOfEnumValues(const EnumDecl *ED) {
 137 |   llvm::SmallSet<llvm::APSInt, 16> EnumValues;
 138 | 
```
- EN: This block continues the implementation with declarations or statements centered on `}`.
- CN: 这一段继续实现，围绕 `}` 展开声明或语句。

### Lines 139-144
```cpp
 139 |   const EnumConstantDecl *LastEnumConstant = nullptr;
 140 |   for (const EnumConstantDecl *Enumerator : ED->enumerators()) {
 141 |     EnumValues.insert(Enumerator->getInitVal());
 142 |     LastEnumConstant = Enumerator;
 143 |   }
 144 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const EnumConstantDecl *LastEnumConstant = nullptr;`.
- CN: 这一段继续实现，围绕 `const EnumConstantDecl *LastEnumConstant = nullptr;` 展开声明或语句。

### Lines 145-151
```cpp
 145 |   if (EnableCountingEnumHeuristic && LastEnumConstant &&
 146 |       isCountingEnumLikeName(LastEnumConstant->getName()) &&
 147 |       llvm::APSInt::isSameValue(LastEnumConstant->getInitVal(),
 148 |                                 llvm::APSInt::get(EnumValues.size() - 1))) {
 149 |     return {EnumValues.size() - 1, LastEnumConstant};
 150 |   }
 151 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 152-161
```cpp
 152 |   return {EnumValues.size(), nullptr};
 153 | }
 154 | 
 155 | void TaggedUnionMemberCountCheck::check(
 156 |     const MatchFinder::MatchResult &Result) {
 157 |   const auto *Root = Result.Nodes.getNodeAs<RecordDecl>(RootMatchBindName);
 158 |   const auto *UnionField =
 159 |       Result.Nodes.getNodeAs<FieldDecl>(UnionMatchBindName);
 160 |   const auto *TagField = Result.Nodes.getNodeAs<FieldDecl>(TagMatchBindName);
 161 | 
```
- EN: Method definitions such as `TaggedUnionMemberCountCheck::check` provide the concrete behavior declared elsewhere.
- CN: 诸如 `TaggedUnionMemberCountCheck::check` 的方法定义给出了前面声明的具体行为。
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 162-167
```cpp
 162 |   assert(Root && "Root is missing!");
 163 |   assert(UnionField && "UnionField is missing!");
 164 |   assert(TagField && "TagField is missing!");
 165 |   if (!Root || !UnionField || !TagField)
 166 |     return;
 167 | 
```
- EN: This block continues the implementation with declarations or statements centered on `assert(Root && "Root is missing!");`.
- CN: 这一段继续实现，围绕 `assert(Root && "Root is missing!");` 展开声明或语句。

### Lines 168-173
```cpp
 168 |   const auto *UnionDef = UnionField->getType()->castAsRecordDecl();
 169 |   const auto *EnumDef = TagField->getType()->castAsEnumDecl();
 170 | 
 171 |   const std::size_t UnionMemberCount = llvm::range_size(UnionDef->fields());
 172 |   auto [TagCount, CountingEnumConstantDecl] = getNumberOfEnumValues(EnumDef);
 173 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const auto *UnionDef = UnionField->getType()->castAsRecordDe`.
- CN: 这一段继续实现，围绕 `const auto *UnionDef = UnionField->getType()->castAsRecordDe` 展开声明或语句。

### Lines 174-183
```cpp
 174 |   if (UnionMemberCount > TagCount) {
 175 |     diag(Root->getLocation(),
 176 |          "tagged union has more data members (%0) than tags (%1)!")
 177 |         << UnionMemberCount << TagCount;
 178 |   } else if (StrictMode && UnionMemberCount < TagCount) {
 179 |     diag(Root->getLocation(),
 180 |          "tagged union has fewer data members (%0) than tags (%1)!")
 181 |         << UnionMemberCount << TagCount;
 182 |   }
 183 | 
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

### Lines 184-192
```cpp
 184 |   if (CountingEnumConstantDecl) {
 185 |     diag(CountingEnumConstantDecl->getLocation(),
 186 |          "assuming that this constant is just an auxiliary value and not "
 187 |          "used for indicating a valid union data member",
 188 |          DiagnosticIDs::Note);
 189 |   }
 190 | }
 191 | 
 192 | } // namespace clang::tidy::bugprone
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

## Key Concepts / 关键概念
- clang-tidy check lifecycle / clang-tidy 检查生命周期
- AST matcher DSL / AST 匹配器 DSL
- diagnostics and fix-its / 诊断与修复建议
- namespace scoping / 命名空间作用域

## Dependencies / 依赖关系
- EN: Direct includes: `TaggedUnionMemberCountCheck.h`, `../utils/OptionsUtils.h`, `clang/ASTMatchers/ASTMatchFinder.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallSet.h`.
- CN: 直接包含依赖: `TaggedUnionMemberCountCheck.h`、`../utils/OptionsUtils.h`、`clang/ASTMatchers/ASTMatchFinder.h`、`llvm/ADT/STLExtras.h`、`llvm/ADT/SmallSet.h`。
- EN: Namespace context: `clang::tidy::bugprone`.
- CN: 命名空间上下文: `clang::tidy::bugprone`。
