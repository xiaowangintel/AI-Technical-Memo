# ForwardDeclarationNamespaceCheck.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `clang-tools-extra/clang-tidy/bugprone/ForwardDeclarationNamespaceCheck.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `ForwardDeclarationNamespaceCheck` by registering AST matchers and producing diagnostics/fix-its.
- 用途 (CN): 通过注册 AST 匹配器并生成诊断/修复建议来实现 `ForwardDeclarationNamespaceCheck`。

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
   9 | #include "ForwardDeclarationNamespaceCheck.h"
  10 | #include "clang/AST/ASTContext.h"
  11 | #include "clang/AST/Decl.h"
  12 | #include "clang/ASTMatchers/ASTMatchFinder.h"
  13 | #include "clang/ASTMatchers/ASTMatchers.h"
  14 | #include <string>
  15 | 
```
- EN: The section imports dependencies such as `ForwardDeclarationNamespaceCheck.h`, `clang/AST/ASTContext.h`, `clang/AST/Decl.h`, `clang/ASTMatchers/ASTMatchFinder.h` needed by this file.
- CN: 本段引入了 `ForwardDeclarationNamespaceCheck.h`、`clang/AST/ASTContext.h`、`clang/AST/Decl.h`、`clang/ASTMatchers/ASTMatchFinder.h` 等依赖，供当前文件使用。

### Lines 16-19
```cpp
  16 | using namespace clang::ast_matchers;
  17 | 
  18 | namespace clang::tidy::bugprone {
  19 | 
```
- EN: Using-directives shorten qualified names so matcher/transformer code reads more directly.
- CN: using 指令缩短了限定名，使匹配器/变换器代码更直接。
- EN: Namespace scopes such as `clang::tidy::bugprone` place the symbols in their intended subsystem.
- CN: 诸如 `clang::tidy::bugprone` 这样的命名空间将符号放入预期的子系统中。

### Lines 20-29
```cpp
  20 | void ForwardDeclarationNamespaceCheck::registerMatchers(MatchFinder *Finder) {
  21 |   // Match all class declarations/definitions *EXCEPT*
  22 |   // 1. implicit classes, e.g. `class A {};` has implicit `class A` inside `A`.
  23 |   // 2. nested classes declared/defined inside another class.
  24 |   // 3. template class declaration, template instantiation or
  25 |   //    specialization (NOTE: extern specialization is filtered out by
  26 |   //    `unless(hasAncestor(cxxRecordDecl()))`).
  27 |   auto IsInSpecialization = hasAncestor(
  28 |       decl(anyOf(cxxRecordDecl(isExplicitTemplateSpecialization()),
  29 |                  functionDecl(isExplicitTemplateSpecialization()))));
```
- EN: This block defines AST matcher registration, i.e. the structural patterns the check will look for.
- CN: 这一段定义 AST 匹配器注册，也就是该检查要查找的结构模式。
- EN: Method definitions such as `ForwardDeclarationNamespaceCheck::registerMatchers` provide the concrete behavior declared elsewhere.
- CN: 诸如 `ForwardDeclarationNamespaceCheck::registerMatchers` 的方法定义给出了前面声明的具体行为。

### Lines 30-38
```cpp
  30 |   Finder->addMatcher(
  31 |       cxxRecordDecl(
  32 |           hasParent(decl(anyOf(namespaceDecl(), translationUnitDecl()))),
  33 |           unless(isImplicit()), unless(hasAncestor(cxxRecordDecl())),
  34 |           unless(isInstantiated()), unless(IsInSpecialization),
  35 |           unless(classTemplateSpecializationDecl()))
  36 |           .bind("record_decl"),
  37 |       this);
  38 | 
```
- EN: This block continues the implementation with declarations or statements centered on `Finder->addMatcher(`.
- CN: 这一段继续实现，围绕 `Finder->addMatcher(` 展开声明或语句。

### Lines 39-44
```cpp
  39 |   // Match all friend declarations. Classes used in friend declarations are not
  40 |   // marked as referenced in AST. We need to record all record classes used in
  41 |   // friend declarations.
  42 |   Finder->addMatcher(friendDecl().bind("friend_decl"), this);
  43 | }
  44 | 
```
- EN: This block continues the implementation with declarations or statements centered on `// Match all friend declarations. Classes used in friend dec`.
- CN: 这一段继续实现，围绕 `// Match all friend declarations. Classes used in friend dec` 展开声明或语句。

### Lines 45-54
```cpp
  45 | void ForwardDeclarationNamespaceCheck::check(
  46 |     const MatchFinder::MatchResult &Result) {
  47 |   if (const auto *RecordDecl =
  48 |           Result.Nodes.getNodeAs<CXXRecordDecl>("record_decl")) {
  49 |     const StringRef DeclName = RecordDecl->getName();
  50 |     if (RecordDecl->isThisDeclarationADefinition()) {
  51 |       DeclNameToDefinitions[DeclName].push_back(RecordDecl);
  52 |     } else {
  53 |       // If a declaration has no definition, the definition could be in another
  54 |       // namespace (a wrong namespace).
```
- EN: Method definitions such as `ForwardDeclarationNamespaceCheck::check` provide the concrete behavior declared elsewhere.
- CN: 诸如 `ForwardDeclarationNamespaceCheck::check` 的方法定义给出了前面声明的具体行为。

### Lines 55-62
```cpp
  55 |       // NOTE: even a declaration does have definition, we still need it to
  56 |       // compare with other declarations.
  57 |       DeclNameToDeclarations[DeclName].push_back(RecordDecl);
  58 |     }
  59 |   } else {
  60 |     const auto *Decl = Result.Nodes.getNodeAs<FriendDecl>("friend_decl");
  61 |     assert(Decl && "Decl is neither record_decl nor friend decl!");
  62 | 
```
- EN: This block continues the implementation with declarations or statements centered on `// NOTE: even a declaration does have definition, we still n`.
- CN: 这一段继续实现，围绕 `// NOTE: even a declaration does have definition, we still n` 展开声明或语句。

### Lines 63-72
```cpp
  63 |     // Classes used in friend declarations are not marked referenced in AST,
  64 |     // so we need to check classes used in friend declarations manually to
  65 |     // reduce the rate of false positive.
  66 |     // For example, in
  67 |     //    \code
  68 |     //      struct A;
  69 |     //      struct B { friend A; };
  70 |     //    \endcode
  71 |     // `A` will not be marked as "referenced" in the AST.
  72 |     if (const TypeSourceInfo *Tsi = Decl->getFriendType())
```
- EN: This block continues the implementation with declarations or statements centered on `// Classes used in friend declarations are not marked refere`.
- CN: 这一段继续实现，围绕 `// Classes used in friend declarations are not marked refere` 展开声明或语句。

### Lines 73-77
```cpp
  73 |       FriendTypes.insert(
  74 |           Tsi->getType()->getCanonicalTypeUnqualified().getTypePtr());
  75 |   }
  76 | }
  77 | 
```
- EN: This block continues the implementation with declarations or statements centered on `FriendTypes.insert(`.
- CN: 这一段继续实现，围绕 `FriendTypes.insert(` 展开声明或语句。

### Lines 78-82
```cpp
  78 | static bool haveSameNamespaceOrTranslationUnit(const CXXRecordDecl *Decl1,
  79 |                                                const CXXRecordDecl *Decl2) {
  80 |   const DeclContext *ParentDecl1 = Decl1->getLexicalParent();
  81 |   const DeclContext *ParentDecl2 = Decl2->getLexicalParent();
  82 | 
```
- EN: This block continues the implementation with declarations or statements centered on `static bool haveSameNamespaceOrTranslationUnit(const CXXReco`.
- CN: 这一段继续实现，围绕 `static bool haveSameNamespaceOrTranslationUnit(const CXXReco` 展开声明或语句。

### Lines 83-92
```cpp
  83 |   // Since we only matched declarations whose parent is Namespace or
  84 |   // TranslationUnit declaration, the parent should be either a translation unit
  85 |   // or namespace.
  86 |   if (ParentDecl1->getDeclKind() == Decl::TranslationUnit ||
  87 |       ParentDecl2->getDeclKind() == Decl::TranslationUnit) {
  88 |     return ParentDecl1 == ParentDecl2;
  89 |   }
  90 |   assert(ParentDecl1->getDeclKind() == Decl::Namespace &&
  91 |          "ParentDecl1 declaration must be a namespace");
  92 |   assert(ParentDecl2->getDeclKind() == Decl::Namespace &&
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 93-98
```cpp
  93 |          "ParentDecl2 declaration must be a namespace");
  94 |   auto *Ns1 = NamespaceDecl::castFromDeclContext(ParentDecl1);
  95 |   auto *Ns2 = NamespaceDecl::castFromDeclContext(ParentDecl2);
  96 |   return Ns1->getFirstDecl() == Ns2->getFirstDecl();
  97 | }
  98 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 99-108
```cpp
  99 | static std::string getNameOfNamespace(const CXXRecordDecl *Decl) {
 100 |   const auto *ParentDecl = Decl->getLexicalParent();
 101 |   if (ParentDecl->getDeclKind() == Decl::TranslationUnit)
 102 |     return "(global)";
 103 |   const auto *NsDecl = cast<NamespaceDecl>(ParentDecl);
 104 |   std::string Ns;
 105 |   llvm::raw_string_ostream OStream(Ns);
 106 |   NsDecl->printQualifiedName(OStream);
 107 |   return Ns.empty() ? "(global)" : Ns;
 108 | }
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 109-118
```cpp
 109 | 
 110 | void ForwardDeclarationNamespaceCheck::onEndOfTranslationUnit() {
 111 |   // Iterate each group of declarations by name.
 112 |   for (const auto &KeyValuePair : DeclNameToDeclarations) {
 113 |     const auto &Declarations = KeyValuePair.second;
 114 |     // If more than 1 declaration exists, we check if all are in the same
 115 |     // namespace.
 116 |     for (const auto *CurDecl : Declarations) {
 117 |       if (CurDecl->hasDefinition() || CurDecl->isReferenced())
 118 |         continue; // Skip forward declarations that are used/referenced.
```
- EN: Method definitions such as `ForwardDeclarationNamespaceCheck::onEndOfTranslationUnit` provide the concrete behavior declared elsewhere.
- CN: 诸如 `ForwardDeclarationNamespaceCheck::onEndOfTranslationUnit` 的方法定义给出了前面声明的具体行为。

### Lines 119-128
```cpp
 119 |       if (FriendTypes.contains(CurDecl->getASTContext()
 120 |                                    .getCanonicalTagType(CurDecl)
 121 |                                    ->getTypePtr())) {
 122 |         continue; // Skip forward declarations referenced as friend.
 123 |       }
 124 |       if (CurDecl->getLocation().isMacroID() ||
 125 |           CurDecl->getLocation().isInvalid()) {
 126 |         continue;
 127 |       }
 128 |       // Compare with all other declarations with the same name.
```
- EN: This block continues the implementation with declarations or statements centered on `if (FriendTypes.contains(CurDecl->getASTContext()`.
- CN: 这一段继续实现，围绕 `if (FriendTypes.contains(CurDecl->getASTContext()` 展开声明或语句。

### Lines 129-138
```cpp
 129 |       for (const auto *Decl : Declarations) {
 130 |         if (Decl == CurDecl)
 131 |           continue; // Don't compare with self.
 132 |         if (!CurDecl->hasDefinition() &&
 133 |             !haveSameNamespaceOrTranslationUnit(CurDecl, Decl)) {
 134 |           diag(CurDecl->getLocation(),
 135 |                "declaration %0 is never referenced, but a declaration with "
 136 |                "the same name found in another namespace '%1'")
 137 |               << CurDecl << getNameOfNamespace(Decl);
 138 |           diag(Decl->getLocation(), "a declaration of %0 is found here",
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

### Lines 139-148
```cpp
 139 |                DiagnosticIDs::Note)
 140 |               << Decl;
 141 |           break; // FIXME: We only generate one warning for each declaration.
 142 |         }
 143 |       }
 144 |       // Check if a definition in another namespace exists.
 145 |       const auto DeclName = CurDecl->getName();
 146 |       auto It = DeclNameToDefinitions.find(DeclName);
 147 |       if (It == DeclNameToDefinitions.end())
 148 |         continue; // No definition in this translation unit, we can skip it.
```
- EN: This block continues the implementation with declarations or statements centered on `DiagnosticIDs::Note)`.
- CN: 这一段继续实现，围绕 `DiagnosticIDs::Note)` 展开声明或语句。

### Lines 149-158
```cpp
 149 |       // Make a warning for each definition with the same name (in other
 150 |       // namespaces).
 151 |       const auto &Definitions = It->second;
 152 |       for (const auto *Def : Definitions) {
 153 |         diag(CurDecl->getLocation(),
 154 |              "no definition found for %0, but a definition with "
 155 |              "the same name %1 found in another namespace '%2'")
 156 |             << CurDecl << Def << getNameOfNamespace(Def);
 157 |         diag(Def->getLocation(), "a definition of %0 is found here",
 158 |              DiagnosticIDs::Note)
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

### Lines 159-165
```cpp
 159 |             << Def;
 160 |       }
 161 |     }
 162 |   }
 163 | }
 164 | 
 165 | } // namespace clang::tidy::bugprone
```
- EN: This block continues the implementation with declarations or statements centered on `<< Def;`.
- CN: 这一段继续实现，围绕 `<< Def;` 展开声明或语句。

## Key Concepts / 关键概念
- AST matcher DSL / AST 匹配器 DSL
- diagnostics and fix-its / 诊断与修复建议
- namespace scoping / 命名空间作用域

## Dependencies / 依赖关系
- EN: Direct includes: `ForwardDeclarationNamespaceCheck.h`, `clang/AST/ASTContext.h`, `clang/AST/Decl.h`, `clang/ASTMatchers/ASTMatchFinder.h`, `clang/ASTMatchers/ASTMatchers.h`, `string`.
- CN: 直接包含依赖: `ForwardDeclarationNamespaceCheck.h`、`clang/AST/ASTContext.h`、`clang/AST/Decl.h`、`clang/ASTMatchers/ASTMatchFinder.h`、`clang/ASTMatchers/ASTMatchers.h`、`string`。
- EN: Namespace context: `clang::tidy::bugprone`.
- CN: 命名空间上下文: `clang::tidy::bugprone`。
