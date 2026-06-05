# Linkage.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/AST/Linkage.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file provides AST-internal utilities for linkage and visibility calculation.
  - **CN**: 声明 Clang AST 数据结构、语义辅助逻辑以及源码级建模支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===----- Linkage.h - Linkage calculation-related utilities ----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/Clang file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/Clang 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-16
```cpp
//
// This file provides AST-internal utilities for linkage and visibility
// calculation.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_CLANG_LIB_AST_LINKAGE_H
#define LLVM_CLANG_LIB_AST_LINKAGE_H

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 17-24
```cpp
#include "clang/AST/ASTFwd.h"
#include "clang/AST/Decl.h"
#include "clang/AST/DeclCXX.h"
#include "clang/AST/Type.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/PointerIntPair.h"
#include <optional>

```
- **EN**: Pulls in the headers needed by this translation unit, including `clang/AST/ASTFwd.h`, `clang/AST/Decl.h`, `clang/AST/DeclCXX.h`, `clang/AST/Type.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `clang/AST/ASTFwd.h`, `clang/AST/Decl.h`, `clang/AST/DeclCXX.h`, `clang/AST/Type.h`。

### Lines 25-31
```cpp
namespace clang {
/// Kinds of LV computation.  The linkage side of the computation is
/// always the same, but different things can change how visibility is
/// computed.
struct LVComputationKind {
  /// The kind of entity whose visibility is ultimately being computed;
  /// visibility computations for types and non-types follow different rules.
```
- **EN**: Introduces declarations for `clang`, `LVComputationKind`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `clang`, `LVComputationKind` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 32-38
```cpp
  LLVM_PREFERRED_TYPE(bool)
  unsigned ExplicitKind : 1;
  /// Whether explicit visibility attributes should be ignored. When set,
  /// visibility may only be restricted by the visibility of template arguments.
  LLVM_PREFERRED_TYPE(bool)
  unsigned IgnoreExplicitVisibility : 1;
  /// Whether all visibility should be ignored. When set, we're only interested
```
- **EN**: Contains supporting implementation details for the surrounding Clang AST subsystem.
- **CN**: 包含周边 Clang AST 子系统所需的辅助实现细节。

### Lines 39-48
```cpp
  /// in computing linkage.
  LLVM_PREFERRED_TYPE(bool)
  unsigned IgnoreAllVisibility : 1;

  static constexpr int NumLVComputationKindBits = 3;

  explicit LVComputationKind(NamedDecl::ExplicitVisibilityKind EK)
      : ExplicitKind(EK), IgnoreExplicitVisibility(false),
        IgnoreAllVisibility(false) {}

```
- **EN**: Implements logic around `LVComputationKind`, `ExplicitKind`, `IgnoreAllVisibility`; this block supports compile-time evaluation or interpreter-style execution; maintains declaration identity, lookup, or linkage bookkeeping.
- **CN**: 围绕 `LVComputationKind`, `ExplicitKind`, `IgnoreAllVisibility` 实现具体逻辑；该代码块支持编译期求值或解释器式执行，并维护声明身份、查找或链接属性簿记。

### Lines 49-59
```cpp
  NamedDecl::ExplicitVisibilityKind getExplicitVisibilityKind() const {
    return static_cast<NamedDecl::ExplicitVisibilityKind>(ExplicitKind);
  }

  bool isTypeVisibility() const {
    return getExplicitVisibilityKind() == NamedDecl::VisibilityForType;
  }
  bool isValueVisibility() const {
    return getExplicitVisibilityKind() == NamedDecl::VisibilityForValue;
  }

```
- **EN**: Implements logic around `getExplicitVisibilityKind`, `ExplicitVisibilityKind>`, `isTypeVisibility`, `isValueVisibility`.
- **CN**: 围绕 `getExplicitVisibilityKind`, `ExplicitVisibilityKind>`, `isTypeVisibility`, `isValueVisibility` 实现具体逻辑。

### Lines 60-67
```cpp
  /// Do an LV computation when we only care about the linkage.
  static LVComputationKind forLinkageOnly() {
    LVComputationKind Result(NamedDecl::VisibilityForValue);
    Result.IgnoreExplicitVisibility = true;
    Result.IgnoreAllVisibility = true;
    return Result;
  }

```
- **EN**: Implements logic around `forLinkageOnly`, `Result`; this block maintains declaration identity, lookup, or linkage bookkeeping.
- **CN**: 围绕 `forLinkageOnly`, `Result` 实现具体逻辑；该代码块维护声明身份、查找或链接属性簿记。

### Lines 68-76
```cpp
  unsigned toBits() {
    unsigned Bits = 0;
    Bits = (Bits << 1) | ExplicitKind;
    Bits = (Bits << 1) | IgnoreExplicitVisibility;
    Bits = (Bits << 1) | IgnoreAllVisibility;
    return Bits;
  }
};

```
- **EN**: Implements logic around `toBits`.
- **CN**: 围绕 `toBits` 实现具体逻辑。

### Lines 77-90
```cpp
class LinkageComputer {
  // We have a cache for repeated linkage/visibility computations. This saves us
  // from exponential behavior in heavily templated code, such as:
  //
  // template <typename T, typename V> struct {};
  // using A = int;
  // using B = Foo<A, A>;
  // using C = Foo<B, B>;
  // using D = Foo<C, C>;
  //
  // The integer represents an LVComputationKind.
  using QueryType =
      llvm::PointerIntPair<const NamedDecl *,
                           LVComputationKind::NumLVComputationKindBits>;
```
- **EN**: Introduces declarations for `LinkageComputer`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LinkageComputer` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 91-104
```cpp
  llvm::SmallDenseMap<QueryType, LinkageInfo, 8> CachedLinkageInfo;

  static QueryType makeCacheKey(const NamedDecl *ND, LVComputationKind Kind) {
    return QueryType(ND, Kind.toBits());
  }

  std::optional<LinkageInfo> lookup(const NamedDecl *ND,
                                    LVComputationKind Kind) const {
    auto Iter = CachedLinkageInfo.find(makeCacheKey(ND, Kind));
    if (Iter == CachedLinkageInfo.end())
      return std::nullopt;
    return Iter->second;
  }

```
- **EN**: Implements logic around `makeCacheKey`, `QueryType`, `lookup`, `find`, and 1 more symbols; this block maintains declaration identity, lookup, or linkage bookkeeping.
- **CN**: 围绕 `makeCacheKey`, `QueryType`, `lookup`, `find`, and 1 more symbols 实现具体逻辑；该代码块维护声明身份、查找或链接属性簿记。

### Lines 105-111
```cpp
  void cache(const NamedDecl *ND, LVComputationKind Kind, LinkageInfo Info) {
    CachedLinkageInfo[makeCacheKey(ND, Kind)] = Info;
  }

  LinkageInfo getLVForTemplateArgumentList(ArrayRef<TemplateArgument> Args,
                                           LVComputationKind computation);

```
- **EN**: Implements logic around `cache`, `makeCacheKey`, `getLVForTemplateArgumentList`.
- **CN**: 围绕 `cache`, `makeCacheKey`, `getLVForTemplateArgumentList` 实现具体逻辑。

### Lines 112-118
```cpp
  LinkageInfo getLVForTemplateArgumentList(const TemplateArgumentList &TArgs,
                                           LVComputationKind computation);

  void mergeTemplateLV(LinkageInfo &LV, const FunctionDecl *fn,
                       const FunctionTemplateSpecializationInfo *specInfo,
                       LVComputationKind computation);

```
- **EN**: Declares APIs around `getLVForTemplateArgumentList`, `mergeTemplateLV`.
- **CN**: 声明与 `getLVForTemplateArgumentList`, `mergeTemplateLV` 相关的 API。

### Lines 119-126
```cpp
  void mergeTemplateLV(LinkageInfo &LV,
                       const ClassTemplateSpecializationDecl *spec,
                       LVComputationKind computation);

  void mergeTemplateLV(LinkageInfo &LV,
                       const VarTemplateSpecializationDecl *spec,
                       LVComputationKind computation);

```
- **EN**: Declares APIs around `mergeTemplateLV`.
- **CN**: 声明与 `mergeTemplateLV` 相关的 API。

### Lines 127-134
```cpp
  LinkageInfo getLVForNamespaceScopeDecl(const NamedDecl *D,
                                         LVComputationKind computation,
                                         bool IgnoreVarTypeLinkage);

  LinkageInfo getLVForClassMember(const NamedDecl *D,
                                  LVComputationKind computation,
                                  bool IgnoreVarTypeLinkage);

```
- **EN**: Declares APIs around `getLVForNamespaceScopeDecl`, `getLVForClassMember`.
- **CN**: 声明与 `getLVForNamespaceScopeDecl`, `getLVForClassMember` 相关的 API。

### Lines 135-142
```cpp
  LinkageInfo getLVForClosure(const DeclContext *DC, Decl *ContextDecl,
                              LVComputationKind computation);

  LinkageInfo getLVForLocalDecl(const NamedDecl *D,
                                LVComputationKind computation);

  LinkageInfo getLVForType(const Type &T, LVComputationKind computation);

```
- **EN**: Declares APIs around `getLVForClosure`, `getLVForLocalDecl`, `getLVForType`; this block maintains declaration identity, lookup, or linkage bookkeeping; queries or canonicalizes Clang type-system state.
- **CN**: 声明与 `getLVForClosure`, `getLVForLocalDecl`, `getLVForType` 相关的 API；该代码块维护声明身份、查找或链接属性簿记，并查询或规范化 Clang 类型系统状态。

### Lines 143-152
```cpp
  LinkageInfo getLVForTemplateParameterList(const TemplateParameterList *Params,
                                            LVComputationKind computation);

  LinkageInfo getLVForValue(const APValue &V, LVComputationKind computation);

public:
  LinkageInfo computeLVForDecl(const NamedDecl *D,
                               LVComputationKind computation,
                               bool IgnoreVarTypeLinkage = false);

```
- **EN**: Declares APIs around `getLVForTemplateParameterList`, `getLVForValue`, `computeLVForDecl`; this block supports compile-time evaluation or interpreter-style execution.
- **CN**: 声明与 `getLVForTemplateParameterList`, `getLVForValue`, `computeLVForDecl` 相关的 API；该代码块支持编译期求值或解释器式执行。

### Lines 153-159
```cpp
  LinkageInfo getLVForDecl(const NamedDecl *D, LVComputationKind computation);

  LinkageInfo computeTypeLinkageInfo(const Type *T);
  LinkageInfo computeTypeLinkageInfo(QualType T) {
    return computeTypeLinkageInfo(T.getTypePtr());
  }

```
- **EN**: Implements logic around `getLVForDecl`, `computeTypeLinkageInfo`; this block queries or canonicalizes Clang type-system state.
- **CN**: 围绕 `getLVForDecl`, `computeTypeLinkageInfo` 实现具体逻辑；该代码块查询或规范化 Clang 类型系统状态。

### Lines 160-168
```cpp
  LinkageInfo getDeclLinkageAndVisibility(const NamedDecl *D);

  LinkageInfo getTypeLinkageAndVisibility(const Type *T);
  LinkageInfo getTypeLinkageAndVisibility(QualType T) {
    return getTypeLinkageAndVisibility(T.getTypePtr());
  }
};
} // namespace clang

```
- **EN**: Introduces declarations for `clang`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `clang` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 169-169
```cpp
#endif
```
- **EN**: Contains supporting implementation details for the surrounding Clang AST subsystem.
- **CN**: 包含周边 Clang AST 子系统所需的辅助实现细节。

## Key Concepts / 关键概念

- **Clang AST infrastructure / Clang AST 基础设施**:
  - **EN**: Explains how Clang stores and manipulates source-level syntax and semantic entities.
  - **CN**: 说明 Clang 如何存储并操作源码级语法与语义实体。
- **Compile-time values / 编译期值**:
  - **EN**: Represents folded expressions, aggregates, and symbolic constant-evaluation results.
  - **CN**: 表示折叠后的表达式、聚合值以及符号化的常量求值结果。
- **Declaration hierarchy / 声明层次结构**:
  - **EN**: Covers declaration nodes, redeclaration chains, and lookup-visible entities.
  - **CN**: 涵盖声明节点、重声明链以及参与查找的实体。
- **Expression semantics / 表达式语义**:
  - **EN**: Represents typed expression nodes, value categories, and semantic queries.
  - **CN**: 表示带类型的表达式节点、值类别以及语义查询。
- **Type-system modeling / 类型系统建模**:
  - **EN**: Captures canonical types, qualifiers, and source-facing type sugar.
  - **CN**: 刻画规范类型、限定符以及面向源码的类型语法糖。
- **Template metadata / 模板元数据**:
  - **EN**: Tracks template parameters, arguments, specializations, and instantiation state.
  - **CN**: 跟踪模板参数、实参、特化以及实例化状态。
- **Attributes / 属性机制**:
  - **EN**: Stores source-level attributes and generated metadata attached to AST entities.
  - **CN**: 存储附着在 AST 实体上的源码属性与生成元数据。
- **Qualified types / 限定类型**:
  - **EN**: Uses `QualType` wrappers to preserve qualifiers and canonical-type access.
  - **CN**: 使用 `QualType` 包装来保留限定符并访问规范类型。

## Dependencies / 依赖关系

- **Direct LLVM/Clang includes / 直接的 LLVM/Clang 包含**: `clang/AST/ASTFwd.h`, `clang/AST/Decl.h`, `clang/AST/DeclCXX.h`, `clang/AST/Type.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/PointerIntPair.h`
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: Clang AST node definitions and semantic data structures / Clang AST 节点定义与语义数据结构 (4), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (2)
