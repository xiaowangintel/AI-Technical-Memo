# ExternalASTMerger.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/AST/ExternalASTMerger.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements the ExternalASTMerger, which vends a combination of ASTs from several different ASTContext/FileManager pairs.
  - **CN**: 实现 Clang AST 数据结构、语义辅助逻辑以及源码级建模支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===- ExternalASTMerger.cpp - Merging External AST Interface ---*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
//  This file implements the ExternalASTMerger, which vends a combination of
//  ASTs from several different ASTContext/FileManager pairs
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/Clang file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/Clang 标准文件头、许可证声明以及文件的高层描述。

### Lines 13-22
```cpp

#include "clang/AST/ASTContext.h"
#include "clang/AST/Decl.h"
#include "clang/AST/DeclCXX.h"
#include "clang/AST/DeclObjC.h"
#include "clang/AST/DeclTemplate.h"
#include "clang/AST/ExternalASTMerger.h"

using namespace clang;

```
- **EN**: Pulls in the headers needed by this translation unit, including `clang/AST/ASTContext.h`, `clang/AST/Decl.h`, `clang/AST/DeclCXX.h`, `clang/AST/DeclObjC.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `clang/AST/ASTContext.h`, `clang/AST/Decl.h`, `clang/AST/DeclCXX.h`, `clang/AST/DeclObjC.h`。

### Lines 23-33
```cpp
namespace {

template <typename T> struct Source {
  T t;
  Source(T t) : t(t) {}
  operator T() { return t; }
  template <typename U = T> U &get() { return t; }
  template <typename U = T> const U &get() const { return t; }
  template <typename U> operator Source<U>() { return Source<U>(t); }
};

```
- **EN**: Introduces declarations for `Source`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `Source` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 34-43
```cpp
typedef std::pair<Source<NamedDecl *>, ASTImporter *> Candidate;

/// For the given DC, return the DC that is safe to perform lookups on.  This is
/// the DC we actually want to work with most of the time.
const DeclContext *CanonicalizeDC(const DeclContext *DC) {
  if (isa<LinkageSpecDecl>(DC))
    return DC->getRedeclContext();
  return DC;
}

```
- **EN**: Implements logic around `CanonicalizeDC`, `isa`, `getRedeclContext`.
- **CN**: 围绕 `CanonicalizeDC`, `isa`, `getRedeclContext` 实现具体逻辑。

### Lines 44-63
```cpp
Source<const DeclContext *>
LookupSameContext(Source<TranslationUnitDecl *> SourceTU, const DeclContext *DC,
                  ASTImporter &ReverseImporter) {
  DC = CanonicalizeDC(DC);
  if (DC->isTranslationUnit()) {
    return SourceTU;
  }
  Source<const DeclContext *> SourceParentDC =
      LookupSameContext(SourceTU, DC->getParent(), ReverseImporter);
  if (!SourceParentDC) {
    // If we couldn't find the parent DC in this TranslationUnit, give up.
    return nullptr;
  }
  auto *ND = cast<NamedDecl>(DC);
  DeclarationName Name = ND->getDeclName();
  auto SourceNameOrErr = ReverseImporter.Import(Name);
  if (!SourceNameOrErr) {
    llvm::consumeError(SourceNameOrErr.takeError());
    return nullptr;
  }
```
- **EN**: Implements logic around `LookupSameContext`, `CanonicalizeDC`, `isTranslationUnit`, `cast`, and 3 more symbols; this block propagates diagnostics or recoverable errors through Clang/LLVM utilities; reconciles entities across AST contexts or translation units.
- **CN**: 围绕 `LookupSameContext`, `CanonicalizeDC`, `isTranslationUnit`, `cast`, and 3 more symbols 实现具体逻辑；该代码块通过 Clang/LLVM 工具传播诊断信息或可恢复错误，并在 AST 上下文或翻译单元之间对齐实体。

### Lines 64-83
```cpp
  Source<DeclarationName> SourceName = *SourceNameOrErr;
  DeclContext::lookup_result SearchResult =
      SourceParentDC.get()->lookup(SourceName.get());

  // There are two cases here. First, we might not find the name.
  // We might also find multiple copies, in which case we have no
  // guarantee that the one we wanted is the one we pick.  (E.g.,
  // if we have two specializations of the same template it is
  // very hard to determine which is the one you want.)
  //
  // The Origins map fixes this problem by allowing the origin to be
  // explicitly recorded, so we trigger that recording by returning
  // nothing (rather than a possibly-inaccurate guess) here.
  if (SearchResult.isSingleResult()) {
    NamedDecl *SearchResultDecl = SearchResult.front();
    if (isa<DeclContext>(SearchResultDecl) &&
        SearchResultDecl->getKind() == DC->getDeclKind())
      return cast<DeclContext>(SearchResultDecl)->getPrimaryContext();
    return nullptr; // This type of lookup is unsupported
  } else {
```
- **EN**: Implements logic around `get`, `isSingleResult`, `front`, `isa`, and 2 more symbols; this block tracks template or constraint-related semantic state; maintains declaration identity, lookup, or linkage bookkeeping; queries or canonicalizes Clang type-system state.
- **CN**: 围绕 `get`, `isSingleResult`, `front`, `isa`, and 2 more symbols 实现具体逻辑；该代码块跟踪模板或约束相关的语义状态，并维护声明身份、查找或链接属性簿记，并查询或规范化 Clang 类型系统状态。

### Lines 84-93
```cpp
    return nullptr;
  }
}

/// A custom implementation of ASTImporter, for ExternalASTMerger's purposes.
///
/// There are several modifications:
///
/// - It enables lazy lookup (via the HasExternalLexicalStorage flag and a few
///   others), which instructs Clang to refer to ExternalASTMerger.  Also, it
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 94-104
```cpp
///   forces MinimalImport to true, which is necessary to make this work.
/// - It maintains a reverse importer for use with names.  This allows lookup of
///   arbitrary names in the source context.
/// - It updates the ExternalASTMerger's origin map as needed whenever a
///   it sees a DeclContext.
class LazyASTImporter : public ASTImporter {
private:
  ExternalASTMerger &Parent;
  ASTImporter Reverse;
  const ExternalASTMerger::OriginMap &FromOrigins;
  /// @see ExternalASTMerger::ImporterSource::Temporary
```
- **EN**: Introduces declarations for `LazyASTImporter`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LazyASTImporter` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 105-124
```cpp
  bool TemporarySource;
  /// Map of imported declarations back to the declarations they originated
  /// from.
  llvm::DenseMap<Decl *, Decl *> ToOrigin;
  /// @see ExternalASTMerger::ImporterSource::Merger
  ExternalASTMerger *SourceMerger;
  llvm::raw_ostream &logs() { return Parent.logs(); }
public:
  LazyASTImporter(ExternalASTMerger &_Parent, ASTContext &ToContext,
                  FileManager &ToFileManager,
                  const ExternalASTMerger::ImporterSource &S,
                  std::shared_ptr<ASTImporterSharedState> SharedState)
      : ASTImporter(ToContext, ToFileManager, S.getASTContext(),
                    S.getFileManager(),
                    /*MinimalImport=*/true, SharedState),
        Parent(_Parent),
        Reverse(S.getASTContext(), S.getFileManager(), ToContext, ToFileManager,
                /*MinimalImport=*/true),
        FromOrigins(S.getOriginMap()), TemporarySource(S.isTemporary()),
        SourceMerger(S.getMerger()) {}
```
- **EN**: Implements logic around `logs`, `LazyASTImporter`, `ASTImporter`, `getFileManager`, and 4 more symbols; this block reconciles entities across AST contexts or translation units; maintains declaration identity, lookup, or linkage bookkeeping.
- **CN**: 围绕 `logs`, `LazyASTImporter`, `ASTImporter`, `getFileManager`, and 4 more symbols 实现具体逻辑；该代码块在 AST 上下文或翻译单元之间对齐实体，并维护声明身份、查找或链接属性簿记。

### Lines 125-144
```cpp

  llvm::Expected<Decl *> ImportImpl(Decl *FromD) override {
    if (!TemporarySource || !SourceMerger)
      return ASTImporter::ImportImpl(FromD);

    // If we get here, then this source is importing from a temporary ASTContext
    // that also has another ExternalASTMerger attached. It could be
    // possible that the current ExternalASTMerger and the temporary ASTContext
    // share a common ImporterSource, which means that the temporary
    // AST could contain declarations that were imported from a source
    // that this ExternalASTMerger can access directly. Instead of importing
    // such declarations from the temporary ASTContext, they should instead
    // be directly imported by this ExternalASTMerger from the original
    // source. This way the ExternalASTMerger can safely do a minimal import
    // without creating incomplete declarations originated from a temporary
    // ASTContext. If we would try to complete such declarations later on, we
    // would fail to do so as their temporary AST could be deleted (which means
    // that the missing parts of the minimally imported declaration in that
    // ASTContext were also deleted).
    //
```
- **EN**: Implements logic around `ImportImpl`; this block reconciles entities across AST contexts or translation units; maintains declaration identity, lookup, or linkage bookkeeping.
- **CN**: 围绕 `ImportImpl` 实现具体逻辑；该代码块在 AST 上下文或翻译单元之间对齐实体，并维护声明身份、查找或链接属性簿记。

### Lines 145-161
```cpp
    // The following code tracks back any declaration that needs to be
    // imported from the temporary ASTContext to a persistent ASTContext.
    // Then the ExternalASTMerger tries to import from the persistent
    // ASTContext directly by using the associated ASTImporter. If that
    // succeeds, this ASTImporter just maps the declarations imported by
    // the other (persistent) ASTImporter to this (temporary) ASTImporter.
    // The steps can be visualized like this:
    //
    //  Target AST <--- 3. Indirect import --- Persistent AST
    //       ^            of persistent decl        ^
    //       |                                      |
    // 1. Current import           2. Tracking back to persistent decl
    // 4. Map persistent decl                       |
    //  & pretend we imported.                      |
    //       |                                      |
    // Temporary AST -------------------------------'

```
- **EN**: Documents the next declarations or records design constraints for the surrounding AST implementation.
- **CN**: 为接下来的声明提供说明，或记录周边 AST 实现的设计约束。

### Lines 162-181
```cpp
    // First, ask the ExternalASTMerger of the source where the temporary
    // declaration originated from.
    Decl *Persistent = SourceMerger->FindOriginalDecl(FromD);
    // FromD isn't from a persistent AST, so just do a normal import.
    if (!Persistent)
      return ASTImporter::ImportImpl(FromD);
    // Now ask the current ExternalASTMerger to try import the persistent
    // declaration into the target.
    ASTContext &PersistentCtx = Persistent->getASTContext();
    ASTImporter &OtherImporter = Parent.ImporterForOrigin(PersistentCtx);
    // Check that we never end up in the current Importer again.
    assert((&PersistentCtx != &getFromContext()) && (&OtherImporter != this) &&
           "Delegated to same Importer?");
    auto DeclOrErr = OtherImporter.Import(Persistent);
    // Errors when importing the persistent decl are treated as if we
    // had errors with importing the temporary decl.
    if (!DeclOrErr)
      return DeclOrErr.takeError();
    Decl *D = *DeclOrErr;
    // Tell the current ASTImporter that this has already been imported
```
- **EN**: Implements logic around `FindOriginalDecl`, `ImportImpl`, `getASTContext`, `ImporterForOrigin`, and 3 more symbols; this block propagates diagnostics or recoverable errors through Clang/LLVM utilities; reconciles entities across AST contexts or translation units; maintains declaration identity, lookup, or linkage bookkeeping.
- **CN**: 围绕 `FindOriginalDecl`, `ImportImpl`, `getASTContext`, `ImporterForOrigin`, and 3 more symbols 实现具体逻辑；该代码块通过 Clang/LLVM 工具传播诊断信息或可恢复错误，并在 AST 上下文或翻译单元之间对齐实体，并维护声明身份、查找或链接属性簿记。

### Lines 182-192
```cpp
    // to prevent any further queries for the temporary decl.
    MapImported(FromD, D);
    return D;
  }

  /// Implements the ASTImporter interface for tracking back a declaration
  /// to its original declaration it came from.
  Decl *GetOriginalDecl(Decl *To) override {
    return ToOrigin.lookup(To);
  }

```
- **EN**: Implements logic around `MapImported`, `GetOriginalDecl`, `lookup`; this block maintains declaration identity, lookup, or linkage bookkeeping.
- **CN**: 围绕 `MapImported`, `GetOriginalDecl`, `lookup` 实现具体逻辑；该代码块维护声明身份、查找或链接属性簿记。

### Lines 193-212
```cpp
  /// Whenever a DeclContext is imported, ensure that ExternalASTSource's origin
  /// map is kept up to date.  Also set the appropriate flags.
  void Imported(Decl *From, Decl *To) override {
    ToOrigin[To] = From;

    if (auto *ToDC = dyn_cast<DeclContext>(To)) {
      const bool LoggingEnabled = Parent.LoggingEnabled();
      if (LoggingEnabled)
        logs() << "(ExternalASTMerger*)" << (void*)&Parent
               << " imported (DeclContext*)" << (void*)ToDC
               << ", (ASTContext*)" << (void*)&getToContext()
               << " from (DeclContext*)" << (void*)llvm::cast<DeclContext>(From)
               << ", (ASTContext*)" << (void*)&getFromContext()
               << "\n";
      Source<DeclContext *> FromDC(
          cast<DeclContext>(From)->getPrimaryContext());
      if (auto It = FromOrigins.find(FromDC);
          It != FromOrigins.end() &&
          Parent.HasImporterForOrigin(*It->second.AST)) {
        if (LoggingEnabled)
```
- **EN**: Implements logic around `Imported`, `dyn_cast`, `LoggingEnabled`, `logs`, and 9 more symbols; this block reconciles entities across AST contexts or translation units; maintains declaration identity, lookup, or linkage bookkeeping.
- **CN**: 围绕 `Imported`, `dyn_cast`, `LoggingEnabled`, `logs`, and 9 more symbols 实现具体逻辑；该代码块在 AST 上下文或翻译单元之间对齐实体，并维护声明身份、查找或链接属性簿记。

### Lines 213-232
```cpp
          logs() << "(ExternalASTMerger*)" << (void *)&Parent
                 << " forced origin (DeclContext*)" << (void *)It->second.DC
                 << ", (ASTContext*)" << (void *)It->second.AST << "\n";
        Parent.ForceRecordOrigin(ToDC, It->second);
      } else {
        if (LoggingEnabled)
          logs() << "(ExternalASTMerger*)" << (void*)&Parent
                 << " maybe recording origin (DeclContext*)" << (void*)FromDC
                 << ", (ASTContext*)" << (void*)&getFromContext()
                 << "\n";
        Parent.MaybeRecordOrigin(ToDC, {FromDC, &getFromContext()});
      }
    }
    if (auto *ToTag = dyn_cast<TagDecl>(To)) {
      ToTag->setHasExternalLexicalStorage();
      ToTag->getPrimaryContext()->setMustBuildLookupTable();
      assert(Parent.CanComplete(ToTag));
    } else if (auto *ToNamespace = dyn_cast<NamespaceDecl>(To)) {
      ToNamespace->setHasExternalVisibleStorage();
      assert(Parent.CanComplete(ToNamespace));
```
- **EN**: Implements logic around `logs`, `origin`, `ForceRecordOrigin`, `getFromContext`, and 6 more symbols.
- **CN**: 围绕 `logs`, `origin`, `ForceRecordOrigin`, `getFromContext`, and 6 more symbols 实现具体逻辑。

### Lines 233-249
```cpp
    } else if (auto *ToContainer = dyn_cast<ObjCContainerDecl>(To)) {
      ToContainer->setHasExternalLexicalStorage();
      ToContainer->getPrimaryContext()->setMustBuildLookupTable();
      assert(Parent.CanComplete(ToContainer));
    }
  }
  ASTImporter &GetReverse() { return Reverse; }
};

bool HasDeclOfSameType(ArrayRef<Candidate> Decls, const Candidate &C) {
  if (isa<FunctionDecl>(C.first.get()))
    return false;
  return llvm::any_of(Decls, [&](const Candidate &D) {
    return C.first.get()->getKind() == D.first.get()->getKind();
  });
}

```
- **EN**: Implements logic around `dyn_cast`, `setHasExternalLexicalStorage`, `getPrimaryContext`, `assert`, and 5 more symbols.
- **CN**: 围绕 `dyn_cast`, `setHasExternalLexicalStorage`, `getPrimaryContext`, `assert`, and 5 more symbols 实现具体逻辑。

### Lines 250-259
```cpp
} // end namespace

ASTImporter &ExternalASTMerger::ImporterForOrigin(ASTContext &OriginContext) {
  for (const std::unique_ptr<ASTImporter> &I : Importers)
    if (&I->getFromContext() == &OriginContext)
      return *I;
  llvm_unreachable("We should have an importer for this origin!");
}

namespace {
```
- **EN**: Implements logic around `ImporterForOrigin`, `getFromContext`, `llvm_unreachable`.
- **CN**: 围绕 `ImporterForOrigin`, `getFromContext`, `llvm_unreachable` 实现具体逻辑。

### Lines 260-273
```cpp
LazyASTImporter &LazyImporterForOrigin(ExternalASTMerger &Merger,
                                   ASTContext &OriginContext) {
  return static_cast<LazyASTImporter &>(
      Merger.ImporterForOrigin(OriginContext));
}
}

bool ExternalASTMerger::HasImporterForOrigin(ASTContext &OriginContext) {
  for (const std::unique_ptr<ASTImporter> &I : Importers)
    if (&I->getFromContext() == &OriginContext)
      return true;
  return false;
}

```
- **EN**: Implements logic around `LazyImporterForOrigin`, `ImporterForOrigin`, `HasImporterForOrigin`, `getFromContext`.
- **CN**: 围绕 `LazyImporterForOrigin`, `ImporterForOrigin`, `HasImporterForOrigin`, `getFromContext` 实现具体逻辑。

### Lines 274-293
```cpp
template <typename CallbackType>
void ExternalASTMerger::ForEachMatchingDC(const DeclContext *DC,
                                          CallbackType Callback) {
  if (auto It = Origins.find(DC); It != Origins.end()) {
    ExternalASTMerger::DCOrigin Origin = It->second;
    LazyASTImporter &Importer = LazyImporterForOrigin(*this, *Origin.AST);
    Callback(Importer, Importer.GetReverse(), Origin.DC);
  } else {
    bool DidCallback = false;
    for (const std::unique_ptr<ASTImporter> &Importer : Importers) {
      Source<TranslationUnitDecl *> SourceTU =
          Importer->getFromContext().getTranslationUnitDecl();
      ASTImporter &Reverse =
          static_cast<LazyASTImporter *>(Importer.get())->GetReverse();
      if (auto SourceDC = LookupSameContext(SourceTU, DC, Reverse)) {
        DidCallback = true;
        if (Callback(*Importer, Reverse, SourceDC))
          break;
      }
    }
```
- **EN**: Implements logic around `ForEachMatchingDC`, `find`, `LazyImporterForOrigin`, `Callback`, and 3 more symbols; this block tracks template or constraint-related semantic state.
- **CN**: 围绕 `ForEachMatchingDC`, `find`, `LazyImporterForOrigin`, `Callback`, and 3 more symbols 实现具体逻辑；该代码块跟踪模板或约束相关的语义状态。

### Lines 294-313
```cpp
    if (!DidCallback && LoggingEnabled())
      logs() << "(ExternalASTMerger*)" << (void*)this
             << " asserting for (DeclContext*)" << (const void*)DC
             << ", (ASTContext*)" << (void*)&Target.AST
             << "\n";
    assert(DidCallback && "Couldn't find a source context matching our DC");
  }
}

void ExternalASTMerger::CompleteType(TagDecl *Tag) {
  assert(Tag->hasExternalLexicalStorage());
  ForEachMatchingDC(Tag, [&](ASTImporter &Forward, ASTImporter &Reverse,
                             Source<const DeclContext *> SourceDC) -> bool {
    auto *SourceTag = const_cast<TagDecl *>(cast<TagDecl>(SourceDC.get()));
    if (SourceTag->hasExternalLexicalStorage())
      SourceTag->getASTContext().getExternalSource()->CompleteType(SourceTag);
    if (!SourceTag->getDefinition())
      return false;
    Forward.MapImported(SourceTag, Tag);
    if (llvm::Error Err = Forward.ImportDefinition(SourceTag))
```
- **EN**: Implements logic around `LoggingEnabled`, `logs`, `assert`, `CompleteType`, and 7 more symbols; this block propagates diagnostics or recoverable errors through Clang/LLVM utilities.
- **CN**: 围绕 `LoggingEnabled`, `logs`, `assert`, `CompleteType`, and 7 more symbols 实现具体逻辑；该代码块通过 Clang/LLVM 工具传播诊断信息或可恢复错误。

### Lines 314-333
```cpp
      llvm::consumeError(std::move(Err));
    Tag->setCompleteDefinition(SourceTag->isCompleteDefinition());
    return true;
  });
}

void ExternalASTMerger::CompleteType(ObjCInterfaceDecl *Interface) {
  assert(Interface->hasExternalLexicalStorage());
  ForEachMatchingDC(
      Interface, [&](ASTImporter &Forward, ASTImporter &Reverse,
                     Source<const DeclContext *> SourceDC) -> bool {
        auto *SourceInterface = const_cast<ObjCInterfaceDecl *>(
            cast<ObjCInterfaceDecl>(SourceDC.get()));
        if (SourceInterface->hasExternalLexicalStorage())
          SourceInterface->getASTContext().getExternalSource()->CompleteType(
              SourceInterface);
        if (!SourceInterface->getDefinition())
          return false;
        Forward.MapImported(SourceInterface, Interface);
        if (llvm::Error Err = Forward.ImportDefinition(SourceInterface))
```
- **EN**: Implements logic around `consumeError`, `setCompleteDefinition`, `CompleteType`, `assert`, and 7 more symbols; this block propagates diagnostics or recoverable errors through Clang/LLVM utilities.
- **CN**: 围绕 `consumeError`, `setCompleteDefinition`, `CompleteType`, `assert`, and 7 more symbols 实现具体逻辑；该代码块通过 Clang/LLVM 工具传播诊断信息或可恢复错误。

### Lines 334-351
```cpp
          llvm::consumeError(std::move(Err));
        return true;
      });
}

bool ExternalASTMerger::CanComplete(DeclContext *Interface) {
  assert(Interface->hasExternalLexicalStorage() ||
         Interface->hasExternalVisibleStorage());
  bool FoundMatchingDC = false;
  ForEachMatchingDC(Interface,
                    [&](ASTImporter &Forward, ASTImporter &Reverse,
                        Source<const DeclContext *> SourceDC) -> bool {
                      FoundMatchingDC = true;
                      return true;
                    });
  return FoundMatchingDC;
}

```
- **EN**: Implements logic around `consumeError`, `CanComplete`, `assert`, `hasExternalVisibleStorage`, and 1 more symbols.
- **CN**: 围绕 `consumeError`, `CanComplete`, `assert`, `hasExternalVisibleStorage`, and 1 more symbols 实现具体逻辑。

### Lines 352-363
```cpp
namespace {
bool IsSameDC(const DeclContext *D1, const DeclContext *D2) {
  if (isa<ObjCContainerDecl>(D1) && isa<ObjCContainerDecl>(D2))
    return true; // There are many cases where Objective-C is ambiguous.
  if (auto *T1 = dyn_cast<TagDecl>(D1))
    if (auto *T2 = dyn_cast<TagDecl>(D2))
      if (T1->getFirstDecl() == T2->getFirstDecl())
        return true;
  return D1 == D2 || D1 == CanonicalizeDC(D2);
}
}

```
- **EN**: Implements logic around `IsSameDC`, `isa`, `dyn_cast`, `getFirstDecl`, and 1 more symbols.
- **CN**: 围绕 `IsSameDC`, `isa`, `dyn_cast`, `getFirstDecl`, and 1 more symbols 实现具体逻辑。

### Lines 364-380
```cpp
void ExternalASTMerger::MaybeRecordOrigin(const DeclContext *ToDC,
                                          DCOrigin Origin) {
  LazyASTImporter &Importer = LazyImporterForOrigin(*this, *Origin.AST);
  ASTImporter &Reverse = Importer.GetReverse();
  Source<const DeclContext *> FoundFromDC =
      LookupSameContext(Origin.AST->getTranslationUnitDecl(), ToDC, Reverse);
  const bool DoRecord = !FoundFromDC || !IsSameDC(FoundFromDC.get(), Origin.DC);
  if (DoRecord)
    RecordOriginImpl(ToDC, Origin, Importer);
  if (LoggingEnabled())
    logs() << "(ExternalASTMerger*)" << (void*)this
             << (DoRecord ? " decided " : " decided NOT")
             << " to record origin (DeclContext*)" << (void*)Origin.DC
             << ", (ASTContext*)" << (void*)&Origin.AST
             << "\n";
}

```
- **EN**: Implements logic around `MaybeRecordOrigin`, `LazyImporterForOrigin`, `GetReverse`, `LookupSameContext`, and 5 more symbols.
- **CN**: 围绕 `MaybeRecordOrigin`, `LazyImporterForOrigin`, `GetReverse`, `LookupSameContext`, and 5 more symbols 实现具体逻辑。

### Lines 381-391
```cpp
void ExternalASTMerger::ForceRecordOrigin(const DeclContext *ToDC,
                                          DCOrigin Origin) {
  RecordOriginImpl(ToDC, Origin, ImporterForOrigin(*Origin.AST));
}

void ExternalASTMerger::RecordOriginImpl(const DeclContext *ToDC, DCOrigin Origin,
                                         ASTImporter &Importer) {
  Origins[ToDC] = Origin;
  Importer.ASTImporter::MapImported(cast<Decl>(Origin.DC), const_cast<Decl*>(cast<Decl>(ToDC)));
}

```
- **EN**: Implements logic around `ForceRecordOrigin`, `RecordOriginImpl`, `MapImported`; this block maintains declaration identity, lookup, or linkage bookkeeping.
- **CN**: 围绕 `ForceRecordOrigin`, `RecordOriginImpl`, `MapImported` 实现具体逻辑；该代码块维护声明身份、查找或链接属性簿记。

### Lines 392-407
```cpp
ExternalASTMerger::ExternalASTMerger(const ImporterTarget &Target,
                                     ArrayRef<ImporterSource> Sources)
    : LogStream(&llvm::nulls()), Target(Target) {
  SharedState = std::make_shared<ASTImporterSharedState>(
      *Target.AST.getTranslationUnitDecl());
  AddSources(Sources);
}

Decl *ExternalASTMerger::FindOriginalDecl(Decl *D) {
  assert(&D->getASTContext() == &Target.AST);
  for (const auto &I : Importers)
    if (auto Result = I->GetOriginalDecl(D))
      return Result;
  return nullptr;
}

```
- **EN**: Implements logic around `ExternalASTMerger`, `LogStream`, `make_shared`, `getTranslationUnitDecl`, and 4 more symbols; this block maintains declaration identity, lookup, or linkage bookkeeping.
- **CN**: 围绕 `ExternalASTMerger`, `LogStream`, `make_shared`, `getTranslationUnitDecl`, and 4 more symbols 实现具体逻辑；该代码块维护声明身份、查找或链接属性簿记。

### Lines 408-417
```cpp
void ExternalASTMerger::AddSources(ArrayRef<ImporterSource> Sources) {
  for (const ImporterSource &S : Sources) {
    assert(&S.getASTContext() != &Target.AST);
    // Check that the associated merger actually imports into the source AST.
    assert(!S.getMerger() || &S.getMerger()->Target.AST == &S.getASTContext());
    Importers.push_back(std::make_unique<LazyASTImporter>(
        *this, Target.AST, Target.FM, S, SharedState));
  }
}

```
- **EN**: Implements logic around `AddSources`, `assert`, `push_back`.
- **CN**: 围绕 `AddSources`, `assert`, `push_back` 实现具体逻辑。

### Lines 418-437
```cpp
void ExternalASTMerger::RemoveSources(ArrayRef<ImporterSource> Sources) {
  if (LoggingEnabled())
    for (const ImporterSource &S : Sources)
      logs() << "(ExternalASTMerger*)" << (void *)this
             << " removing source (ASTContext*)" << (void *)&S.getASTContext()
             << "\n";
  llvm::erase_if(Importers,
                 [&Sources](std::unique_ptr<ASTImporter> &Importer) -> bool {
                   for (const ImporterSource &S : Sources) {
                     if (&Importer->getFromContext() == &S.getASTContext())
                       return true;
                   }
                   return false;
                 });
  for (OriginMap::iterator OI = Origins.begin(), OE = Origins.end(); OI != OE; ) {
    std::pair<const DeclContext *, DCOrigin> Origin = *OI;
    bool Erase = false;
    for (const ImporterSource &S : Sources) {
      if (&S.getASTContext() == Origin.second.AST) {
        Erase = true;
```
- **EN**: Implements logic around `RemoveSources`, `LoggingEnabled`, `logs`, `source`, and 4 more symbols.
- **CN**: 围绕 `RemoveSources`, `LoggingEnabled`, `logs`, `source`, and 4 more symbols 实现具体逻辑。

### Lines 438-447
```cpp
        break;
      }
    }
    if (Erase)
      OI = Origins.erase(OI);
    else
      ++OI;
  }
}

```
- **EN**: Implements logic around `erase`.
- **CN**: 围绕 `erase` 实现具体逻辑。

### Lines 448-459
```cpp
template <typename DeclTy>
static bool importSpecializations(DeclTy *D, ASTImporter *Importer) {
  for (auto *Spec : D->specializations()) {
    auto ImportedSpecOrError = Importer->Import(Spec);
    if (!ImportedSpecOrError) {
      llvm::consumeError(ImportedSpecOrError.takeError());
      return true;
    }
  }
  return false;
}

```
- **EN**: Implements logic around `importSpecializations`, `specializations`, `Import`, `consumeError`; this block propagates diagnostics or recoverable errors through Clang/LLVM utilities; reconciles entities across AST contexts or translation units; tracks template or constraint-related semantic state.
- **CN**: 围绕 `importSpecializations`, `specializations`, `Import`, `consumeError` 实现具体逻辑；该代码块通过 Clang/LLVM 工具传播诊断信息或可恢复错误，并在 AST 上下文或翻译单元之间对齐实体，并跟踪模板或约束相关的语义状态。

### Lines 460-472
```cpp
/// Imports specializations from template declarations that can be specialized.
static bool importSpecializationsIfNeeded(Decl *D, ASTImporter *Importer) {
  if (!isa<TemplateDecl>(D))
    return false;
  if (auto *FunctionTD = dyn_cast<FunctionTemplateDecl>(D))
    return importSpecializations(FunctionTD, Importer);
  else if (auto *ClassTD = dyn_cast<ClassTemplateDecl>(D))
    return importSpecializations(ClassTD, Importer);
  else if (auto *VarTD = dyn_cast<VarTemplateDecl>(D))
    return importSpecializations(VarTD, Importer);
  return false;
}

```
- **EN**: Implements logic around `importSpecializationsIfNeeded`, `isa`, `dyn_cast`, `importSpecializations`; this block tracks template or constraint-related semantic state; maintains declaration identity, lookup, or linkage bookkeeping.
- **CN**: 围绕 `importSpecializationsIfNeeded`, `isa`, `dyn_cast`, `importSpecializations` 实现具体逻辑；该代码块跟踪模板或约束相关的语义状态，并维护声明身份、查找或链接属性簿记。

### Lines 473-483
```cpp
bool ExternalASTMerger::FindExternalVisibleDeclsByName(
    const DeclContext *DC, DeclarationName Name,
    const DeclContext *OriginalDC) {
  llvm::SmallVector<NamedDecl *, 1> Decls;
  llvm::SmallVector<Candidate, 4> Candidates;

  auto FilterFoundDecl = [&Candidates](const Candidate &C) {
   if (!HasDeclOfSameType(Candidates, C))
     Candidates.push_back(C);
  };

```
- **EN**: Implements logic around `FindExternalVisibleDeclsByName`, `HasDeclOfSameType`, `push_back`.
- **CN**: 围绕 `FindExternalVisibleDeclsByName`, `HasDeclOfSameType`, `push_back` 实现具体逻辑。

### Lines 484-499
```cpp
  ForEachMatchingDC(DC,
                    [&](ASTImporter &Forward, ASTImporter &Reverse,
                        Source<const DeclContext *> SourceDC) -> bool {
                      auto FromNameOrErr = Reverse.Import(Name);
                      if (!FromNameOrErr) {
                        llvm::consumeError(FromNameOrErr.takeError());
                        return false;
                      }
                      DeclContextLookupResult Result =
                          SourceDC.get()->lookup(*FromNameOrErr);
                      for (NamedDecl *FromD : Result) {
                        FilterFoundDecl(std::make_pair(FromD, &Forward));
                      }
                      return false;
                    });

```
- **EN**: Implements logic around `ForEachMatchingDC`, `Import`, `consumeError`, `get`, and 1 more symbols; this block propagates diagnostics or recoverable errors through Clang/LLVM utilities; reconciles entities across AST contexts or translation units; maintains declaration identity, lookup, or linkage bookkeeping.
- **CN**: 围绕 `ForEachMatchingDC`, `Import`, `consumeError`, `get`, and 1 more symbols 实现具体逻辑；该代码块通过 Clang/LLVM 工具传播诊断信息或可恢复错误，并在 AST 上下文或翻译单元之间对齐实体，并维护声明身份、查找或链接属性簿记。

### Lines 500-519
```cpp
  if (Candidates.empty())
    return false;

  Decls.reserve(Candidates.size());
  for (const Candidate &C : Candidates) {
    Decl *LookupRes = C.first.get();
    ASTImporter *Importer = C.second;
    auto NDOrErr = Importer->Import(LookupRes);
    NamedDecl *ND = cast<NamedDecl>(llvm::cantFail(std::move(NDOrErr)));
    assert(ND);
    // If we don't import specialization, they are not available via lookup
    // because the lookup result is imported TemplateDecl and it does not
    // reference its specializations until they are imported explicitly.
    bool IsSpecImportFailed =
        importSpecializationsIfNeeded(LookupRes, Importer);
    assert(!IsSpecImportFailed);
    (void)IsSpecImportFailed;
    Decls.push_back(ND);
  }
  SetExternalVisibleDeclsForName(DC, Name, Decls);
```
- **EN**: Implements logic around `empty`, `reserve`, `get`, `Import`, and 5 more symbols; this block reconciles entities across AST contexts or translation units; tracks template or constraint-related semantic state; maintains declaration identity, lookup, or linkage bookkeeping.
- **CN**: 围绕 `empty`, `reserve`, `get`, `Import`, and 5 more symbols 实现具体逻辑；该代码块在 AST 上下文或翻译单元之间对齐实体，并跟踪模板或约束相关的语义状态，并维护声明身份、查找或链接属性簿记。

### Lines 520-539
```cpp
  return true;
}

void ExternalASTMerger::FindExternalLexicalDecls(
    const DeclContext *DC, llvm::function_ref<bool(Decl::Kind)> IsKindWeWant,
    SmallVectorImpl<Decl *> &Result) {
  ForEachMatchingDC(DC, [&](ASTImporter &Forward, ASTImporter &Reverse,
                            Source<const DeclContext *> SourceDC) -> bool {
    for (const Decl *SourceDecl : SourceDC.get()->decls()) {
      if (IsKindWeWant(SourceDecl->getKind())) {
        auto ImportedDeclOrErr = Forward.Import(SourceDecl);
        if (ImportedDeclOrErr)
          assert(!(*ImportedDeclOrErr) ||
                 IsSameDC((*ImportedDeclOrErr)->getDeclContext(), DC));
        else
          llvm::consumeError(ImportedDeclOrErr.takeError());
      }
    }
    return false;
  });
```
- **EN**: Implements logic around `FindExternalLexicalDecls`, `function_ref`, `ForEachMatchingDC`, `get`, and 5 more symbols; this block propagates diagnostics or recoverable errors through Clang/LLVM utilities; reconciles entities across AST contexts or translation units; maintains declaration identity, lookup, or linkage bookkeeping.
- **CN**: 围绕 `FindExternalLexicalDecls`, `function_ref`, `ForEachMatchingDC`, `get`, and 5 more symbols 实现具体逻辑；该代码块通过 Clang/LLVM 工具传播诊断信息或可恢复错误，并在 AST 上下文或翻译单元之间对齐实体，并维护声明身份、查找或链接属性簿记。

### Lines 540-540
```cpp
}
```
- **EN**: Contains supporting implementation details for the surrounding Clang AST subsystem.
- **CN**: 包含周边 Clang AST 子系统所需的辅助实现细节。

## Key Concepts / 关键概念

- **Clang AST infrastructure / Clang AST 基础设施**:
  - **EN**: Explains how Clang stores and manipulates source-level syntax and semantic entities.
  - **CN**: 说明 Clang 如何存储并操作源码级语法与语义实体。
- **AST context ownership / AST 上下文所有权**:
  - **EN**: Centralizes allocation, uniquing, and lifetime management for AST objects.
  - **CN**: 集中管理 AST 对象的分配、唯一化与生命周期。
- **Declaration hierarchy / 声明层次结构**:
  - **EN**: Covers declaration nodes, redeclaration chains, and lookup-visible entities.
  - **CN**: 涵盖声明节点、重声明链以及参与查找的实体。
- **Type-system modeling / 类型系统建模**:
  - **EN**: Captures canonical types, qualifiers, and source-facing type sugar.
  - **CN**: 刻画规范类型、限定符以及面向源码的类型语法糖。
- **Template metadata / 模板元数据**:
  - **EN**: Tracks template parameters, arguments, specializations, and instantiation state.
  - **CN**: 跟踪模板参数、实参、特化以及实例化状态。
- **Cross-AST importing / 跨 AST 导入**:
  - **EN**: Moves or recreates nodes between different AST contexts while preserving semantics.
  - **CN**: 在不同 AST 上下文之间移动或重建节点并保持语义。
- **Source locations / 源码位置**:
  - **EN**: Tracks source ranges, spelling locations, and mapping back to original files.
  - **CN**: 跟踪源码范围、拼写位置以及回溯到原始文件的映射。
- **AST importer / AST 导入器**:
  - **EN**: Moves declarations and types between different AST contexts while repairing references.
  - **CN**: 在不同 AST 上下文之间迁移声明和类型，并修复引用关系。
- **Declaration names / 声明名称**:
  - **EN**: Stores lookup-oriented name forms used by declarations and templates.
  - **CN**: 存储供声明与模板查找使用的名称形式。

## Dependencies / 依赖关系

- **Direct LLVM/Clang includes / 直接的 LLVM/Clang 包含**: `clang/AST/ASTContext.h`, `clang/AST/Decl.h`, `clang/AST/DeclCXX.h`, `clang/AST/DeclObjC.h`, `clang/AST/DeclTemplate.h`, `clang/AST/ExternalASTMerger.h`
- **Subsystem categories / 子系统类别**: Clang AST node definitions and semantic data structures / Clang AST 节点定义与语义数据结构 (6)
