# VirtualNearMissCheck.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `clang-tools-extra/clang-tidy/bugprone/VirtualNearMissCheck.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `ASTContext` by registering AST matchers and producing diagnostics/fix-its.
- 用途 (CN): 通过注册 AST 匹配器并生成诊断/修复建议来实现 `ASTContext`。

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
   9 | #include "VirtualNearMissCheck.h"
  10 | #include "clang/AST/ASTContext.h"
  11 | #include "clang/AST/CXXInheritance.h"
  12 | #include "clang/ASTMatchers/ASTMatchFinder.h"
  13 | #include "clang/Lex/Lexer.h"
  14 | 
```
- EN: The section imports dependencies such as `VirtualNearMissCheck.h`, `clang/AST/ASTContext.h`, `clang/AST/CXXInheritance.h`, `clang/ASTMatchers/ASTMatchFinder.h` needed by this file.
- CN: 本段引入了 `VirtualNearMissCheck.h`、`clang/AST/ASTContext.h`、`clang/AST/CXXInheritance.h`、`clang/ASTMatchers/ASTMatchFinder.h` 等依赖，供当前文件使用。

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
  19 | namespace {
  20 | AST_MATCHER(CXXMethodDecl, isStatic) { return Node.isStatic(); }
  21 | 
  22 | AST_MATCHER(CXXMethodDecl, isOverloadedOperator) {
  23 |   return Node.isOverloadedOperator();
  24 | }
  25 | } // namespace
  26 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 27-31
```cpp
  27 | /// Finds out if the given method overrides some method.
  28 | static bool isOverrideMethod(const CXXMethodDecl *MD) {
  29 |   return MD->size_overridden_methods() > 0 || MD->hasAttr<OverrideAttr>();
  30 | }
  31 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 32-41
```cpp
  32 | /// Checks whether the return types are covariant, according to
  33 | /// C++[class.virtual]p7.
  34 | ///
  35 | /// Similar with clang::Sema::CheckOverridingFunctionReturnType.
  36 | /// \returns true if the return types of BaseMD and DerivedMD are covariant.
  37 | static bool checkOverridingFunctionReturnType(const ASTContext *Context,
  38 |                                               const CXXMethodDecl *BaseMD,
  39 |                                               const CXXMethodDecl *DerivedMD) {
  40 |   const QualType BaseReturnTy = BaseMD->getType()
  41 |                                     ->castAs<FunctionType>()
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 42-48
```cpp
  42 |                                     ->getReturnType()
  43 |                                     .getCanonicalType();
  44 |   const QualType DerivedReturnTy = DerivedMD->getType()
  45 |                                        ->castAs<FunctionType>()
  46 |                                        ->getReturnType()
  47 |                                        .getCanonicalType();
  48 | 
```
- EN: This block continues the implementation with declarations or statements centered on `->getReturnType()`.
- CN: 这一段继续实现，围绕 `->getReturnType()` 展开声明或语句。

### Lines 49-55
```cpp
  49 |   if (DerivedReturnTy->isDependentType() || BaseReturnTy->isDependentType())
  50 |     return false;
  51 | 
  52 |   // Check if return types are identical.
  53 |   if (ASTContext::hasSameType(DerivedReturnTy, BaseReturnTy))
  54 |     return true;
  55 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 56-62
```cpp
  56 |   /// Check if the return types are covariant.
  57 | 
  58 |   // Both types must be pointers or references to classes.
  59 |   if (!(BaseReturnTy->isPointerType() && DerivedReturnTy->isPointerType()) &&
  60 |       !(BaseReturnTy->isReferenceType() && DerivedReturnTy->isReferenceType()))
  61 |     return false;
  62 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 63-68
```cpp
  63 |   /// BTy is the class type in return type of BaseMD. For example,
  64 |   ///    B* Base::md()
  65 |   /// While BRD is the declaration of B.
  66 |   const QualType DTy = DerivedReturnTy->getPointeeType().getCanonicalType();
  67 |   const QualType BTy = BaseReturnTy->getPointeeType().getCanonicalType();
  68 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 69-73
```cpp
  69 |   const CXXRecordDecl *DRD = DTy->getAsCXXRecordDecl();
  70 |   const CXXRecordDecl *BRD = BTy->getAsCXXRecordDecl();
  71 |   if (DRD == nullptr || BRD == nullptr)
  72 |     return false;
  73 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 74-79
```cpp
  74 |   if (!DRD->hasDefinition() || !BRD->hasDefinition())
  75 |     return false;
  76 | 
  77 |   if (DRD == BRD)
  78 |     return true;
  79 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 80-84
```cpp
  80 |   if (!ASTContext::hasSameUnqualifiedType(DTy, BTy)) {
  81 |     // Begin checking whether the conversion from D to B is valid.
  82 |     CXXBasePaths Paths(/*FindAmbiguities=*/true, /*RecordPaths=*/true,
  83 |                        /*DetectVirtual=*/false);
  84 | 
```
- EN: This block continues the implementation with declarations or statements centered on `if (!ASTContext::hasSameUnqualifiedType(DTy, BTy)) {`.
- CN: 这一段继续实现，围绕 `if (!ASTContext::hasSameUnqualifiedType(DTy, BTy)) {` 展开声明或语句。

### Lines 85-88
```cpp
  85 |     // Check whether D is derived from B, and fill in a CXXBasePaths object.
  86 |     if (!DRD->isDerivedFrom(BRD, Paths))
  87 |       return false;
  88 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 89-93
```cpp
  89 |     // Check ambiguity.
  90 |     if (Paths.isAmbiguous(
  91 |             ASTContext::getCanonicalType(BTy).getUnqualifiedType()))
  92 |       return false;
  93 | 
```
- EN: Method definitions such as `ASTContext::getCanonicalType` provide the concrete behavior declared elsewhere.
- CN: 诸如 `ASTContext::getCanonicalType` 的方法定义给出了前面声明的具体行为。
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 94-103
```cpp
  94 |     // Check accessibility.
  95 |     // FIXME: We currently only support checking if B is accessible base class
  96 |     // of D, or D is the same class which DerivedMD is in.
  97 |     const bool IsItself =
  98 |         DRD->getCanonicalDecl() == DerivedMD->getParent()->getCanonicalDecl();
  99 |     bool HasPublicAccess = false;
 100 |     for (const auto &Path : Paths)
 101 |       if (Path.Access == AS_public)
 102 |         HasPublicAccess = true;
 103 |     if (!HasPublicAccess && !IsItself)
```
- EN: This block continues the implementation with declarations or statements centered on `// Check accessibility.`.
- CN: 这一段继续实现，围绕 `// Check accessibility.` 展开声明或语句。

### Lines 104-107
```cpp
 104 |       return false;
 105 |     // End checking conversion from D to B.
 106 |   }
 107 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 108-112
```cpp
 108 |   // Both pointers or references should have the same cv-qualification.
 109 |   if (DerivedReturnTy.getLocalCVRQualifiers() !=
 110 |       BaseReturnTy.getLocalCVRQualifiers())
 111 |     return false;
 112 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 113-117
```cpp
 113 |   // The class type D should have the same cv-qualification as or less
 114 |   // cv-qualification than the class type B.
 115 |   if (DTy.isMoreQualifiedThan(BTy, *Context))
 116 |     return false;
 117 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 118-127
```cpp
 118 |   return true;
 119 | }
 120 | 
 121 | /// \returns decayed type for arrays and functions.
 122 | static QualType getDecayedType(QualType Type) {
 123 |   if (const auto *Decayed = Type->getAs<DecayedType>())
 124 |     return Decayed->getDecayedType();
 125 |   return Type;
 126 | }
 127 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 128-135
```cpp
 128 | /// \returns true if the param types are the same.
 129 | static bool checkParamTypes(const CXXMethodDecl *BaseMD,
 130 |                             const CXXMethodDecl *DerivedMD) {
 131 |   const unsigned NumParamA = BaseMD->getNumParams();
 132 |   const unsigned NumParamB = DerivedMD->getNumParams();
 133 |   if (NumParamA != NumParamB)
 134 |     return false;
 135 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 136-143
```cpp
 136 |   for (unsigned I = 0; I < NumParamA; I++)
 137 |     if (getDecayedType(BaseMD->getParamDecl(I)->getType().getCanonicalType()) !=
 138 |         getDecayedType(
 139 |             DerivedMD->getParamDecl(I)->getType().getCanonicalType()))
 140 |       return false;
 141 |   return true;
 142 | }
 143 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 144-151
```cpp
 144 | /// \returns true if derived method can override base method except for the
 145 | /// name.
 146 | static bool checkOverrideWithoutName(const ASTContext *Context,
 147 |                                      const CXXMethodDecl *BaseMD,
 148 |                                      const CXXMethodDecl *DerivedMD) {
 149 |   if (BaseMD->isStatic() != DerivedMD->isStatic())
 150 |     return false;
 151 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 152-161
```cpp
 152 |   if (BaseMD->getType() == DerivedMD->getType())
 153 |     return true;
 154 | 
 155 |   // Now the function types are not identical. Then check if the return types
 156 |   // are covariant and if the param types are the same.
 157 |   if (!checkOverridingFunctionReturnType(Context, BaseMD, DerivedMD))
 158 |     return false;
 159 |   return checkParamTypes(BaseMD, DerivedMD);
 160 | }
 161 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 162-171
```cpp
 162 | /// Check whether BaseMD overrides DerivedMD.
 163 | ///
 164 | /// Prerequisite: the class which BaseMD is in should be a base class of that
 165 | /// DerivedMD is in.
 166 | static bool checkOverrideByDerivedMethod(const CXXMethodDecl *BaseMD,
 167 |                                          const CXXMethodDecl *DerivedMD) {
 168 |   for (CXXMethodDecl::method_iterator I = DerivedMD->begin_overridden_methods(),
 169 |                                       E = DerivedMD->end_overridden_methods();
 170 |        I != E; ++I) {
 171 |     const CXXMethodDecl *OverriddenMD = *I;
```
- EN: This block continues the implementation with declarations or statements centered on `/// Check whether BaseMD overrides DerivedMD.`.
- CN: 这一段继续实现，围绕 `/// Check whether BaseMD overrides DerivedMD.` 展开声明或语句。

### Lines 172-175
```cpp
 172 |     if (BaseMD->getCanonicalDecl() == OverriddenMD->getCanonicalDecl())
 173 |       return true;
 174 |   }
 175 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 176-184
```cpp
 176 |   return false;
 177 | }
 178 | 
 179 | bool VirtualNearMissCheck::isPossibleToBeOverridden(
 180 |     const CXXMethodDecl *BaseMD) {
 181 |   auto [Iter, Inserted] = PossibleMap.try_emplace(BaseMD);
 182 |   if (!Inserted)
 183 |     return Iter->second;
 184 | 
```
- EN: Method definitions such as `VirtualNearMissCheck::isPossibleToBeOverridden` provide the concrete behavior declared elsewhere.
- CN: 诸如 `VirtualNearMissCheck::isPossibleToBeOverridden` 的方法定义给出了前面声明的具体行为。
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 185-192
```cpp
 185 |   const bool IsPossible =
 186 |       !BaseMD->isImplicit() && !isa<CXXConstructorDecl>(BaseMD) &&
 187 |       !isa<CXXDestructorDecl>(BaseMD) && BaseMD->isVirtual() &&
 188 |       !BaseMD->isOverloadedOperator() && !isa<CXXConversionDecl>(BaseMD);
 189 |   Iter->second = IsPossible;
 190 |   return IsPossible;
 191 | }
 192 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 193-199
```cpp
 193 | bool VirtualNearMissCheck::isOverriddenByDerivedClass(
 194 |     const CXXMethodDecl *BaseMD, const CXXRecordDecl *DerivedRD) {
 195 |   const std::pair Key(BaseMD, DerivedRD);
 196 |   auto Iter = OverriddenMap.find(Key);
 197 |   if (Iter != OverriddenMap.end())
 198 |     return Iter->second;
 199 | 
```
- EN: Method definitions such as `VirtualNearMissCheck::isOverriddenByDerivedClass` provide the concrete behavior declared elsewhere.
- CN: 诸如 `VirtualNearMissCheck::isOverriddenByDerivedClass` 的方法定义给出了前面声明的具体行为。
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 200-204
```cpp
 200 |   bool IsOverridden = false;
 201 |   for (const CXXMethodDecl *DerivedMD : DerivedRD->methods()) {
 202 |     if (!isOverrideMethod(DerivedMD))
 203 |       continue;
 204 | 
```
- EN: This block continues the implementation with declarations or statements centered on `bool IsOverridden = false;`.
- CN: 这一段继续实现，围绕 `bool IsOverridden = false;` 展开声明或语句。

### Lines 205-213
```cpp
 205 |     if (checkOverrideByDerivedMethod(BaseMD, DerivedMD)) {
 206 |       IsOverridden = true;
 207 |       break;
 208 |     }
 209 |   }
 210 |   OverriddenMap[Key] = IsOverridden;
 211 |   return IsOverridden;
 212 | }
 213 | 
```
- EN: Return statements here hand the constructed rule, value, or decision back to the caller/framework.
- CN: 这里的 return 语句把构造好的规则、值或决策返回给调用方/框架。

### Lines 214-223
```cpp
 214 | void VirtualNearMissCheck::registerMatchers(MatchFinder *Finder) {
 215 |   Finder->addMatcher(
 216 |       cxxMethodDecl(
 217 |           unless(anyOf(isOverride(), isImplicit(), cxxConstructorDecl(),
 218 |                        cxxDestructorDecl(), cxxConversionDecl(), isStatic(),
 219 |                        isOverloadedOperator())))
 220 |           .bind("method"),
 221 |       this);
 222 | }
 223 | 
```
- EN: This block defines AST matcher registration, i.e. the structural patterns the check will look for.
- CN: 这一段定义 AST 匹配器注册，也就是该检查要查找的结构模式。
- EN: Method definitions such as `VirtualNearMissCheck::registerMatchers` provide the concrete behavior declared elsewhere.
- CN: 诸如 `VirtualNearMissCheck::registerMatchers` 的方法定义给出了前面声明的具体行为。

### Lines 224-227
```cpp
 224 | void VirtualNearMissCheck::check(const MatchFinder::MatchResult &Result) {
 225 |   const auto *DerivedMD = Result.Nodes.getNodeAs<CXXMethodDecl>("method");
 226 |   assert(DerivedMD);
 227 | 
```
- EN: Method definitions such as `VirtualNearMissCheck::check` provide the concrete behavior declared elsewhere.
- CN: 诸如 `VirtualNearMissCheck::check` 的方法定义给出了前面声明的具体行为。

### Lines 228-232
```cpp
 228 |   const ASTContext *Context = Result.Context;
 229 | 
 230 |   const auto *DerivedRD = DerivedMD->getParent()->getDefinition();
 231 |   assert(DerivedRD);
 232 | 
```
- EN: This block continues the implementation with declarations or statements centered on `const ASTContext *Context = Result.Context;`.
- CN: 这一段继续实现，围绕 `const ASTContext *Context = Result.Context;` 展开声明或语句。

### Lines 233-238
```cpp
 233 |   for (const auto &BaseSpec : DerivedRD->bases()) {
 234 |     if (const auto *BaseRD = BaseSpec.getType()->getAsCXXRecordDecl()) {
 235 |       for (const auto *BaseMD : BaseRD->methods()) {
 236 |         if (!isPossibleToBeOverridden(BaseMD))
 237 |           continue;
 238 | 
```
- EN: This block continues the implementation with declarations or statements centered on `for (const auto &BaseSpec : DerivedRD->bases()) {`.
- CN: 这一段继续实现，围绕 `for (const auto &BaseSpec : DerivedRD->bases()) {` 展开声明或语句。

### Lines 239-248
```cpp
 239 |         if (isOverriddenByDerivedClass(BaseMD, DerivedRD))
 240 |           continue;
 241 | 
 242 |         const unsigned EditDistance = BaseMD->getName().edit_distance(
 243 |             DerivedMD->getName(), EditDistanceThreshold);
 244 |         if (EditDistance > 0 && EditDistance <= EditDistanceThreshold) {
 245 |           if (checkOverrideWithoutName(Context, BaseMD, DerivedMD)) {
 246 |             // A "virtual near miss" is found.
 247 |             auto Range = CharSourceRange::getTokenRange(
 248 |                 SourceRange(DerivedMD->getLocation()));
```
- EN: This block continues the implementation with declarations or statements centered on `if (isOverriddenByDerivedClass(BaseMD, DerivedRD))`.
- CN: 这一段继续实现，围绕 `if (isOverriddenByDerivedClass(BaseMD, DerivedRD))` 展开声明或语句。

### Lines 249-258
```cpp
 249 | 
 250 |             const bool ApplyFix = !BaseMD->isTemplateInstantiation() &&
 251 |                                   !DerivedMD->isTemplateInstantiation();
 252 |             auto Diag =
 253 |                 diag(DerivedMD->getBeginLoc(),
 254 |                      "method '%0' has a similar name and the same signature as "
 255 |                      "virtual method '%1'; did you mean to override it?")
 256 |                 << DerivedMD->getQualifiedNameAsString()
 257 |                 << BaseMD->getQualifiedNameAsString();
 258 |             if (ApplyFix)
```
- EN: Diagnostics are emitted here, usually together with fix-it hints or contextual messages.
- CN: 这里发出诊断信息，通常还会附带修复建议或上下文消息。

### Lines 259-267
```cpp
 259 |               Diag << FixItHint::CreateReplacement(Range, BaseMD->getName());
 260 |           }
 261 |         }
 262 |       }
 263 |     }
 264 |   }
 265 | }
 266 | 
 267 | } // namespace clang::tidy::bugprone
```
- EN: Method definitions such as `FixItHint::CreateReplacement` provide the concrete behavior declared elsewhere.
- CN: 诸如 `FixItHint::CreateReplacement` 的方法定义给出了前面声明的具体行为。

## Key Concepts / 关键概念
- AST matcher DSL / AST 匹配器 DSL
- diagnostics and fix-its / 诊断与修复建议
- namespace scoping / 命名空间作用域

## Dependencies / 依赖关系
- EN: Direct includes: `VirtualNearMissCheck.h`, `clang/AST/ASTContext.h`, `clang/AST/CXXInheritance.h`, `clang/ASTMatchers/ASTMatchFinder.h`, `clang/Lex/Lexer.h`.
- CN: 直接包含依赖: `VirtualNearMissCheck.h`、`clang/AST/ASTContext.h`、`clang/AST/CXXInheritance.h`、`clang/ASTMatchers/ASTMatchFinder.h`、`clang/Lex/Lexer.h`。
- EN: Namespace context: `clang::tidy::bugprone`.
- CN: 命名空间上下文: `clang::tidy::bugprone`。
