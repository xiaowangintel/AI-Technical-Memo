# CrtpConstructorAccessibilityCheck.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `clang-tools-extra/clang-tidy/bugprone/CrtpConstructorAccessibilityCheck.cpp`
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

### Lines 9-12
```cpp
   9 | #include "CrtpConstructorAccessibilityCheck.h"
  10 | #include "../utils/LexerUtils.h"
  11 | #include "clang/ASTMatchers/ASTMatchFinder.h"
  12 | 
```
- EN: The section imports dependencies such as `CrtpConstructorAccessibilityCheck.h`, `../utils/LexerUtils.h`, `clang/ASTMatchers/ASTMatchFinder.h` needed by this file.
- CN: 本段引入了 `CrtpConstructorAccessibilityCheck.h`、`../utils/LexerUtils.h`、`clang/ASTMatchers/ASTMatchFinder.h` 等依赖，供当前文件使用。

### Lines 13-16
```cpp
  13 | using namespace clang::ast_matchers;
  14 | 
  15 | namespace clang::tidy::bugprone {
  16 | 
```
- EN: Using-directives shorten qualified names so matcher/transformer code reads more directly.
- CN: using 指令缩短了限定名，使匹配器/变换器代码更直接。
- EN: Namespace scopes such as `clang::tidy::bugprone` place the symbols in their intended subsystem.
- CN: 诸如 `clang::tidy::bugprone` 这样的命名空间将符号放入预期的子系统中。

### Lines 17-22
```cpp
  17 | static bool hasPrivateConstructor(const CXXRecordDecl *RD) {
  18 |   return llvm::any_of(RD->ctors(), [](const CXXConstructorDecl *Ctor) {
  19 |     return Ctor->getAccess() == AS_private;
  20 |   });
  21 | }
  22 | 
```
- EN: Method definitions such as `llvm::any_of` provide the concrete behavior declared elsewhere.
- CN: 诸如 `llvm::any_of` 的方法定义给出了前面声明的具体行为。
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 23-29
```cpp
  23 | static bool isDerivedParameterBefriended(const CXXRecordDecl *CRTP,
  24 |                                          const NamedDecl *Param) {
  25 |   return llvm::any_of(CRTP->friends(), [&](const FriendDecl *Friend) {
  26 |     const TypeSourceInfo *const FriendType = Friend->getFriendType();
  27 |     if (!FriendType)
  28 |       return false;
  29 | 
```
- EN: Method definitions such as `llvm::any_of` provide the concrete behavior declared elsewhere.
- CN: 诸如 `llvm::any_of` 的方法定义给出了前面声明的具体行为。
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 30-36
```cpp
  30 |     const auto *const TTPT =
  31 |         dyn_cast<TemplateTypeParmType>(FriendType->getType());
  32 | 
  33 |     return TTPT && TTPT->getDecl() == Param;
  34 |   });
  35 | }
  36 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 37-43
```cpp
  37 | static bool isDerivedClassBefriended(const CXXRecordDecl *CRTP,
  38 |                                      const CXXRecordDecl *Derived) {
  39 |   return llvm::any_of(CRTP->friends(), [&](const FriendDecl *Friend) {
  40 |     const TypeSourceInfo *const FriendType = Friend->getFriendType();
  41 |     if (!FriendType)
  42 |       return false;
  43 | 
```
- EN: Method definitions such as `llvm::any_of` provide the concrete behavior declared elsewhere.
- CN: 诸如 `llvm::any_of` 的方法定义给出了前面声明的具体行为。
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 44-48
```cpp
  44 |     return declaresSameEntity(FriendType->getType()->getAsCXXRecordDecl(),
  45 |                               Derived);
  46 |   });
  47 | }
  48 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 49-58
```cpp
  49 | static const NamedDecl *
  50 | getDerivedParameter(const ClassTemplateSpecializationDecl *CRTP,
  51 |                     const CXXRecordDecl *Derived) {
  52 |   size_t Idx = 0;
  53 |   const bool AnyOf = llvm::any_of(
  54 |       CRTP->getTemplateArgs().asArray(), [&](const TemplateArgument &Arg) {
  55 |         ++Idx;
  56 |         return Arg.getKind() == TemplateArgument::Type &&
  57 |                declaresSameEntity(Arg.getAsType()->getAsCXXRecordDecl(),
  58 |                                   Derived);
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 59-66
```cpp
  59 |       });
  60 | 
  61 |   return AnyOf ? CRTP->getSpecializedTemplate()
  62 |                      ->getTemplateParameters()
  63 |                      ->getParam(Idx - 1)
  64 |                : nullptr;
  65 | }
  66 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 67-71
```cpp
  67 | static std::vector<FixItHint>
  68 | hintMakeCtorPrivate(const CXXConstructorDecl *Ctor,
  69 |                     const std::string &OriginalAccess) {
  70 |   std::vector<FixItHint> Hints;
  71 | 
```
- EN: This block continues the implementation with declarations or statements centered on `static std::vector<FixItHint>`.
- CN: 这一段继续实现，围绕 `static std::vector<FixItHint>` 展开声明或语句。

### Lines 72-81
```cpp
  72 |   Hints.emplace_back(FixItHint::CreateInsertion(
  73 |       Ctor->getBeginLoc().getLocWithOffset(-1), "private:\n"));
  74 | 
  75 |   const ASTContext &ASTCtx = Ctor->getASTContext();
  76 |   const SourceLocation CtorEndLoc =
  77 |       Ctor->isExplicitlyDefaulted()
  78 |           ? utils::lexer::findNextTerminator(Ctor->getEndLoc(),
  79 |                                              ASTCtx.getSourceManager(),
  80 |                                              ASTCtx.getLangOpts())
  81 |           : Ctor->getEndLoc();
```
- EN: This block continues the implementation with declarations or statements centered on `Hints.emplace_back(FixItHint::CreateInsertion(`.
- CN: 这一段继续实现，围绕 `Hints.emplace_back(FixItHint::CreateInsertion(` 展开声明或语句。

### Lines 82-87
```cpp
  82 |   Hints.emplace_back(FixItHint::CreateInsertion(
  83 |       CtorEndLoc.getLocWithOffset(1), '\n' + OriginalAccess + ':' + '\n'));
  84 | 
  85 |   return Hints;
  86 | }
  87 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 88-97
```cpp
  88 | void CrtpConstructorAccessibilityCheck::registerMatchers(MatchFinder *Finder) {
  89 |   Finder->addMatcher(
  90 |       classTemplateSpecializationDecl(
  91 |           decl().bind("crtp"),
  92 |           hasAnyTemplateArgument(refersToType(recordType(hasDeclaration(
  93 |               cxxRecordDecl(
  94 |                   isDerivedFrom(cxxRecordDecl(equalsBoundNode("crtp"))))
  95 |                   .bind("derived")))))),
  96 |       this);
  97 | }
```
- EN: This block defines AST matcher registration, i.e. the structural patterns the check will look for.
- CN: 这一段定义 AST 匹配器注册，也就是该检查要查找的结构模式。
- EN: Method definitions such as `CrtpConstructorAccessibilityCheck::registerMatchers` provide the concrete behavior declared elsewhere.
- CN: 诸如 `CrtpConstructorAccessibilityCheck::registerMatchers` 的方法定义给出了前面声明的具体行为。

### Lines 98-106
```cpp
  98 | 
  99 | void CrtpConstructorAccessibilityCheck::check(
 100 |     const MatchFinder::MatchResult &Result) {
 101 |   const auto *CRTPInstantiation =
 102 |       Result.Nodes.getNodeAs<ClassTemplateSpecializationDecl>("crtp");
 103 |   const auto *DerivedRecord = Result.Nodes.getNodeAs<CXXRecordDecl>("derived");
 104 |   const CXXRecordDecl *CRTPDeclaration =
 105 |       CRTPInstantiation->getSpecializedTemplate()->getTemplatedDecl();
 106 | 
```
- EN: Method definitions such as `CrtpConstructorAccessibilityCheck::check` provide the concrete behavior declared elsewhere.
- CN: 诸如 `CrtpConstructorAccessibilityCheck::check` 的方法定义给出了前面声明的具体行为。

### Lines 107-112
```cpp
 107 |   if (!CRTPDeclaration->hasDefinition())
 108 |     return;
 109 | 
 110 |   const auto *DerivedTemplateParameter =
 111 |       getDerivedParameter(CRTPInstantiation, DerivedRecord);
 112 | 
```
- EN: This block continues the implementation with declarations or statements centered on `if (!CRTPDeclaration->hasDefinition())`.
- CN: 这一段继续实现，围绕 `if (!CRTPDeclaration->hasDefinition())` 展开声明或语句。

### Lines 113-120
```cpp
 113 |   assert(DerivedTemplateParameter &&
 114 |          "No template parameter corresponds to the derived class of the CRTP.");
 115 | 
 116 |   const bool NeedsFriend =
 117 |       !isDerivedParameterBefriended(CRTPDeclaration,
 118 |                                     DerivedTemplateParameter) &&
 119 |       !isDerivedClassBefriended(CRTPDeclaration, DerivedRecord);
 120 | 
```
- EN: This block continues the implementation with declarations or statements centered on `assert(DerivedTemplateParameter &&`.
- CN: 这一段继续实现，围绕 `assert(DerivedTemplateParameter &&` 展开声明或语句。

### Lines 121-124
```cpp
 121 |   const FixItHint HintFriend = FixItHint::CreateInsertion(
 122 |       CRTPDeclaration->getBraceRange().getEnd(),
 123 |       "friend " + DerivedTemplateParameter->getNameAsString() + ';' + '\n');
 124 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const FixItHint HintFriend = FixItHint::CreateInsertion(`.
- CN: 这一段继续实现，围绕 `const FixItHint HintFriend = FixItHint::CreateInsertion(` 展开声明或语句。

### Lines 125-131
```cpp
 125 |   if (hasPrivateConstructor(CRTPDeclaration) && NeedsFriend) {
 126 |     diag(CRTPDeclaration->getLocation(),
 127 |          "the CRTP cannot be constructed from the derived class; consider "
 128 |          "declaring the derived class as friend")
 129 |         << HintFriend;
 130 |   }
 131 | 
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

### Lines 132-137
```cpp
 132 |   auto WithFriendHintIfNeeded = [&](const DiagnosticBuilder &Diag,
 133 |                                     bool NeedsFriend) {
 134 |     if (NeedsFriend)
 135 |       Diag << HintFriend;
 136 |   };
 137 | 
```
- EN: This block continues the implementation with declarations or statements centered on `auto WithFriendHintIfNeeded = [&](const DiagnosticBuilder &D`.
- CN: 这一段继续实现，围绕 `auto WithFriendHintIfNeeded = [&](const DiagnosticBuilder &D` 展开声明或语句。

### Lines 138-147
```cpp
 138 |   if (!CRTPDeclaration->hasUserDeclaredConstructor()) {
 139 |     const bool IsStruct = CRTPDeclaration->isStruct();
 140 | 
 141 |     WithFriendHintIfNeeded(
 142 |         diag(CRTPDeclaration->getLocation(),
 143 |              "the implicit default constructor of the CRTP is publicly "
 144 |              "accessible; consider making it private%select{| and declaring "
 145 |              "the derived class as friend}0")
 146 |             << NeedsFriend
 147 |             << FixItHint::CreateInsertion(
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。
- EN: Method definitions such as `FixItHint::CreateInsertion` provide the concrete behavior declared elsewhere.
- CN: 诸如 `FixItHint::CreateInsertion` 的方法定义给出了前面声明的具体行为。

### Lines 148-155
```cpp
 148 |                    CRTPDeclaration->getBraceRange().getBegin().getLocWithOffset(
 149 |                        1),
 150 |                    (IsStruct ? "\nprivate:\n" : "\n") +
 151 |                        CRTPDeclaration->getNameAsString() + "() = default;\n" +
 152 |                        (IsStruct ? "public:\n" : "")),
 153 |         NeedsFriend);
 154 |   }
 155 | 
```
- EN: This block continues the implementation with declarations or statements centered on `CRTPDeclaration->getBraceRange().getBegin().getLocWithOffset`.
- CN: 这一段继续实现，围绕 `CRTPDeclaration->getBraceRange().getBegin().getLocWithOffset` 展开声明或语句。

### Lines 156-159
```cpp
 156 |   for (auto &&Ctor : CRTPDeclaration->ctors()) {
 157 |     if (Ctor->getAccess() == AS_private || Ctor->isDeleted())
 158 |       continue;
 159 | 
```
- EN: This block continues the implementation with declarations or statements centered on `for (auto &&Ctor : CRTPDeclaration->ctors()) {`.
- CN: 这一段继续实现，围绕 `for (auto &&Ctor : CRTPDeclaration->ctors()) {` 展开声明或语句。

### Lines 160-169
```cpp
 160 |     const bool IsPublic = Ctor->getAccess() == AS_public;
 161 |     const std::string Access = IsPublic ? "public" : "protected";
 162 | 
 163 |     WithFriendHintIfNeeded(
 164 |         diag(Ctor->getLocation(),
 165 |              "%0 constructor allows the CRTP to be %select{inherited "
 166 |              "from|constructed}1 as a regular template class; consider making "
 167 |              "it private%select{| and declaring the derived class as friend}2")
 168 |             << Access << IsPublic << NeedsFriend
 169 |             << hintMakeCtorPrivate(Ctor, Access),
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

### Lines 170-173
```cpp
 170 |         NeedsFriend);
 171 |   }
 172 | }
 173 | 
```
- EN: This block continues the implementation with declarations or statements centered on `NeedsFriend);`.
- CN: 这一段继续实现，围绕 `NeedsFriend);` 展开声明或语句。

### Lines 174-178
```cpp
 174 | bool CrtpConstructorAccessibilityCheck::isLanguageVersionSupported(
 175 |     const LangOptions &LangOpts) const {
 176 |   return LangOpts.CPlusPlus11;
 177 | }
 178 | } // namespace clang::tidy::bugprone
```
- EN: This logic gates the check on a specific language mode so it only runs when the feature is available.
- CN: 这里按特定语言模式进行门控，确保检查只在相应特性可用时运行。
- EN: Method definitions such as `CrtpConstructorAccessibilityCheck::isLanguageVersionSupported` provide the concrete behavior declared elsewhere.
- CN: 诸如 `CrtpConstructorAccessibilityCheck::isLanguageVersionSupported` 的方法定义给出了前面声明的具体行为。
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

## Key Concepts / 关键概念
- AST matcher DSL / AST 匹配器 DSL
- diagnostics and fix-its / 诊断与修复建议
- language-version gating / 语言版本门控
- namespace scoping / 命名空间作用域

## Dependencies / 依赖关系
- EN: Direct includes: `CrtpConstructorAccessibilityCheck.h`, `../utils/LexerUtils.h`, `clang/ASTMatchers/ASTMatchFinder.h`.
- CN: 直接包含依赖: `CrtpConstructorAccessibilityCheck.h`、`../utils/LexerUtils.h`、`clang/ASTMatchers/ASTMatchFinder.h`。
- EN: Namespace context: `clang::tidy::bugprone`.
- CN: 命名空间上下文: `clang::tidy::bugprone`。
