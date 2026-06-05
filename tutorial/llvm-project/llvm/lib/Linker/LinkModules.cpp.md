# LinkModules.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/Linker/LinkModules.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements the LLVM module linker.
  - **CN**: 实现 LLVM IR 模块链接、符号解析与诊断支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- lib/Linker/LinkModules.cpp - Module Linker Implementation ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the LLVM module linker.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件的高层描述。

### Lines 12-23
```cpp

#include "LinkDiagnosticInfo.h"
#include "llvm-c/Linker.h"
#include "llvm/ADT/SetVector.h"
#include "llvm/IR/Comdat.h"
#include "llvm/IR/GlobalValue.h"
#include "llvm/IR/LLVMContext.h"
#include "llvm/IR/Module.h"
#include "llvm/Linker/Linker.h"
#include "llvm/Support/Error.h"
using namespace llvm;

```
- **EN**: Pulls in the headers needed by this translation unit, including `LinkDiagnosticInfo.h`, `llvm-c/Linker.h`, `llvm/ADT/SetVector.h`, `llvm/IR/Comdat.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `LinkDiagnosticInfo.h`, `llvm-c/Linker.h`, `llvm/ADT/SetVector.h`, `llvm/IR/Comdat.h`。

### Lines 24-33
```cpp
namespace {

enum class LinkFrom { Dst, Src, Both };

/// This is an implementation class for the LinkModules function, which is the
/// entrypoint for this file.
class ModuleLinker {
  IRMover &Mover;
  std::unique_ptr<Module> SrcM;

```
- **EN**: Introduces declarations for `LinkFrom`, `for`, `ModuleLinker`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `LinkFrom`, `for`, `ModuleLinker` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 34-43
```cpp
  SetVector<GlobalValue *> ValuesToLink;

  /// For symbol clashes, prefer those from Src.
  unsigned Flags;

  /// List of global value names that should be internalized.
  StringSet<> Internalize;

  /// Function that will perform the actual internalization. The reason for a
  /// callback is that the linker cannot call internalizeModule without
```
- **EN**: Contains supporting implementation details for the surrounding LLVM library component.
- **CN**: 包含周边 LLVM 库组件所需的辅助实现细节。

### Lines 44-55
```cpp
  /// creating a circular dependency between IPO and the linker.
  std::function<void(Module &, const StringSet<> &)> InternalizeCallback;

  /// Used as the callback for lazy linking.
  /// The mover has just hit GV and we have to decide if it, and other members
  /// of the same comdat, should be linked. Every member to be linked is passed
  /// to Add.
  void addLazyFor(GlobalValue &GV, const IRMover::ValueAdder &Add);

  bool shouldOverrideFromSrc() { return Flags & Linker::OverrideFromSrc; }
  bool shouldLinkOnlyNeeded() { return Flags & Linker::LinkOnlyNeeded; }

```
- **EN**: Implements logic around `function`, `addLazyFor`, `shouldOverrideFromSrc`, `shouldLinkOnlyNeeded`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `function`, `addLazyFor`, `shouldOverrideFromSrc`, `shouldLinkOnlyNeeded` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 56-75
```cpp
  bool shouldLinkFromSource(bool &LinkFromSrc, const GlobalValue &Dest,
                            const GlobalValue &Src);

  /// Should we have mover and linker error diag info?
  bool emitError(const Twine &Message) {
    SrcM->getContext().diagnose(LinkDiagnosticInfo(DS_Error, Message));
    return true;
  }

  bool getComdatLeader(Module &M, StringRef ComdatName,
                       const GlobalVariable *&GVar);
  bool computeResultingSelectionKind(StringRef ComdatName,
                                     Comdat::SelectionKind Src,
                                     Comdat::SelectionKind Dst,
                                     Comdat::SelectionKind &Result,
                                     LinkFrom &From);
  DenseMap<const Comdat *, std::pair<Comdat::SelectionKind, LinkFrom>>
      ComdatsChosen;
  bool getComdatResult(const Comdat *SrcC, Comdat::SelectionKind &SK,
                       LinkFrom &From);
```
- **EN**: Implements logic around `shouldLinkFromSource`, `emitError`, `getContext`, `getComdatLeader`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities; emits or serializes data to an external representation; coordinates cross-module linking or optimization state.
- **CN**: 围绕 `shouldLinkFromSource`, `emitError`, `getContext`, `getComdatLeader`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并把数据输出或序列化为外部表示，并协调跨模块链接或优化状态。

### Lines 76-87
```cpp
  // Keep track of the lazy linked global members of each comdat in source.
  DenseMap<const Comdat *, std::vector<GlobalValue *>> LazyComdatMembers;

  /// Given a global in the source module, return the global in the
  /// destination module that is being linked to, if any.
  GlobalValue *getLinkedToGlobal(const GlobalValue *SrcGV) {
    Module &DstM = Mover.getModule();
    // If the source has no name it can't link.  If it has local linkage,
    // there is no name match-up going on.
    if (!SrcGV->hasName() || GlobalValue::isLocalLinkage(SrcGV->getLinkage()))
      return nullptr;

```
- **EN**: Implements logic around `getLinkedToGlobal`, `getModule`, `hasName`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `getLinkedToGlobal`, `getModule`, `hasName` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 88-97
```cpp
    // Otherwise see if we have a match in the destination module's symtab.
    GlobalValue *DGV = DstM.getNamedValue(SrcGV->getName());
    if (!DGV)
      return nullptr;

    // If we found a global with the same name in the dest module, but it has
    // internal linkage, we are really not doing any linkage here.
    if (DGV->hasLocalLinkage())
      return nullptr;

```
- **EN**: Implements logic around `getNamedValue`, `hasLocalLinkage`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `getNamedValue`, `hasLocalLinkage` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 98-108
```cpp
    // Otherwise, we do in fact link to the destination global.
    return DGV;
  }

  /// Drop GV if it is a member of a comdat that we are dropping.
  /// This can happen with COFF's largest selection kind.
  void dropReplacedComdat(GlobalValue &GV,
                          const DenseSet<const Comdat *> &ReplacedDstComdats);

  bool linkIfNeeded(GlobalValue &GV, SmallVectorImpl<GlobalValue *> &GVToClone);

```
- **EN**: Implements logic around `dropReplacedComdat`, `linkIfNeeded`; this block applies object-format-specific rules; coordinates cross-module linking or optimization state.
- **CN**: 围绕 `dropReplacedComdat`, `linkIfNeeded` 实现具体逻辑；该代码块应用目标文件格式专用规则，并协调跨模块链接或优化状态。

### Lines 109-119
```cpp
public:
  ModuleLinker(IRMover &Mover, std::unique_ptr<Module> SrcM, unsigned Flags,
               std::function<void(Module &, const StringSet<> &)>
                   InternalizeCallback = {})
      : Mover(Mover), SrcM(std::move(SrcM)), Flags(Flags),
        InternalizeCallback(std::move(InternalizeCallback)) {}

  bool run();
};
} // namespace

```
- **EN**: Implements logic around `ModuleLinker`, `function`, `Mover`, `InternalizeCallback`, and 1 more symbols; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `ModuleLinker`, `function`, `Mover`, `InternalizeCallback`, and 1 more symbols 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 120-130
```cpp
static GlobalValue::VisibilityTypes
getMinVisibility(GlobalValue::VisibilityTypes A,
                 GlobalValue::VisibilityTypes B) {
  if (A == GlobalValue::HiddenVisibility || B == GlobalValue::HiddenVisibility)
    return GlobalValue::HiddenVisibility;
  if (A == GlobalValue::ProtectedVisibility ||
      B == GlobalValue::ProtectedVisibility)
    return GlobalValue::ProtectedVisibility;
  return GlobalValue::DefaultVisibility;
}

```
- **EN**: Implements logic around `getMinVisibility`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `getMinVisibility` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 131-141
```cpp
bool ModuleLinker::getComdatLeader(Module &M, StringRef ComdatName,
                                   const GlobalVariable *&GVar) {
  const GlobalValue *GVal = M.getNamedValue(ComdatName);
  if (const auto *GA = dyn_cast_or_null<GlobalAlias>(GVal)) {
    GVal = GA->getAliaseeObject();
    if (!GVal)
      // We cannot resolve the size of the aliasee yet.
      return emitError("Linking COMDATs named '" + ComdatName +
                       "': COMDAT key involves incomputable alias size.");
  }

```
- **EN**: Implements logic around `getComdatLeader`, `getNamedValue`, `dyn_cast_or_null`, `getAliaseeObject`, and 1 more symbols; this block emits or serializes data to an external representation; coordinates cross-module linking or optimization state.
- **CN**: 围绕 `getComdatLeader`, `getNamedValue`, `dyn_cast_or_null`, `getAliaseeObject`, and 1 more symbols 实现具体逻辑；该代码块把数据输出或序列化为外部表示，并协调跨模块链接或优化状态。

### Lines 142-161
```cpp
  GVar = dyn_cast_or_null<GlobalVariable>(GVal);
  if (!GVar)
    return emitError(
        "Linking COMDATs named '" + ComdatName +
        "': GlobalVariable required for data dependent selection!");

  return false;
}

bool ModuleLinker::computeResultingSelectionKind(StringRef ComdatName,
                                                 Comdat::SelectionKind Src,
                                                 Comdat::SelectionKind Dst,
                                                 Comdat::SelectionKind &Result,
                                                 LinkFrom &From) {
  Module &DstM = Mover.getModule();
  // The ability to mix Comdat::SelectionKind::Any with
  // Comdat::SelectionKind::Largest is a behavior that comes from COFF.
  bool DstAnyOrLargest = Dst == Comdat::SelectionKind::Any ||
                         Dst == Comdat::SelectionKind::Largest;
  bool SrcAnyOrLargest = Src == Comdat::SelectionKind::Any ||
```
- **EN**: Implements logic around `dyn_cast_or_null`, `emitError`, `computeResultingSelectionKind`, `getModule`; this block emits or serializes data to an external representation; applies object-format-specific rules; coordinates cross-module linking or optimization state.
- **CN**: 围绕 `dyn_cast_or_null`, `emitError`, `computeResultingSelectionKind`, `getModule` 实现具体逻辑；该代码块把数据输出或序列化为外部表示，并应用目标文件格式专用规则，并协调跨模块链接或优化状态。

### Lines 162-175
```cpp
                         Src == Comdat::SelectionKind::Largest;
  if (DstAnyOrLargest && SrcAnyOrLargest) {
    if (Dst == Comdat::SelectionKind::Largest ||
        Src == Comdat::SelectionKind::Largest)
      Result = Comdat::SelectionKind::Largest;
    else
      Result = Comdat::SelectionKind::Any;
  } else if (Src == Dst) {
    Result = Dst;
  } else {
    return emitError("Linking COMDATs named '" + ComdatName +
                     "': invalid selection kinds!");
  }

```
- **EN**: Implements logic around `emitError`; this block emits or serializes data to an external representation; coordinates cross-module linking or optimization state.
- **CN**: 围绕 `emitError` 实现具体逻辑；该代码块把数据输出或序列化为外部表示，并协调跨模块链接或优化状态。

### Lines 176-192
```cpp
  switch (Result) {
  case Comdat::SelectionKind::Any:
    // Go with Dst.
    From = LinkFrom::Dst;
    break;
  case Comdat::SelectionKind::NoDeduplicate:
    From = LinkFrom::Both;
    break;
  case Comdat::SelectionKind::ExactMatch:
  case Comdat::SelectionKind::Largest:
  case Comdat::SelectionKind::SameSize: {
    const GlobalVariable *DstGV;
    const GlobalVariable *SrcGV;
    if (getComdatLeader(DstM, ComdatName, DstGV) ||
        getComdatLeader(*SrcM, ComdatName, SrcGV))
      return true;

```
- **EN**: Implements logic around `getComdatLeader`; this block uses `switch`-style dispatch; coordinates cross-module linking or optimization state.
- **CN**: 围绕 `getComdatLeader` 实现具体逻辑；该代码块使用 `switch` 风格分派，并协调跨模块链接或优化状态。

### Lines 193-212
```cpp
    const DataLayout &DstDL = DstM.getDataLayout();
    const DataLayout &SrcDL = SrcM->getDataLayout();
    uint64_t DstSize = DstGV->getGlobalSize(DstDL);
    uint64_t SrcSize = SrcGV->getGlobalSize(SrcDL);
    if (Result == Comdat::SelectionKind::ExactMatch) {
      if (SrcGV->getInitializer() != DstGV->getInitializer())
        return emitError("Linking COMDATs named '" + ComdatName +
                         "': ExactMatch violated!");
      From = LinkFrom::Dst;
    } else if (Result == Comdat::SelectionKind::Largest) {
      From = SrcSize > DstSize ? LinkFrom::Src : LinkFrom::Dst;
    } else if (Result == Comdat::SelectionKind::SameSize) {
      if (SrcSize != DstSize)
        return emitError("Linking COMDATs named '" + ComdatName +
                         "': SameSize violated!");
      From = LinkFrom::Dst;
    } else {
      llvm_unreachable("unknown selection kind");
    }
    break;
```
- **EN**: Implements logic around `getDataLayout`, `getGlobalSize`, `getInitializer`, `emitError`, and 1 more symbols; this block emits or serializes data to an external representation; coordinates cross-module linking or optimization state.
- **CN**: 围绕 `getDataLayout`, `getGlobalSize`, `getInitializer`, `emitError`, and 1 more symbols 实现具体逻辑；该代码块把数据输出或序列化为外部表示，并协调跨模块链接或优化状态。

### Lines 213-227
```cpp
  }
  }

  return false;
}

bool ModuleLinker::getComdatResult(const Comdat *SrcC,
                                   Comdat::SelectionKind &Result,
                                   LinkFrom &From) {
  Module &DstM = Mover.getModule();
  Comdat::SelectionKind SSK = SrcC->getSelectionKind();
  StringRef ComdatName = SrcC->getName();
  Module::ComdatSymTabType &ComdatSymTab = DstM.getComdatSymbolTable();
  Module::ComdatSymTabType::iterator DstCI = ComdatSymTab.find(ComdatName);

```
- **EN**: Implements logic around `getComdatResult`, `getModule`, `getSelectionKind`, `getName`, and 2 more symbols; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `getComdatResult`, `getModule`, `getSelectionKind`, `getName`, and 2 more symbols 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 228-239
```cpp
  if (DstCI == ComdatSymTab.end()) {
    // Use the comdat if it is only available in one of the modules.
    From = LinkFrom::Src;
    Result = SSK;
    return false;
  }

  const Comdat *DstC = &DstCI->second;
  Comdat::SelectionKind DSK = DstC->getSelectionKind();
  return computeResultingSelectionKind(ComdatName, SSK, DSK, Result, From);
}

```
- **EN**: Implements logic around `end`, `getSelectionKind`, `computeResultingSelectionKind`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `end`, `getSelectionKind`, `computeResultingSelectionKind` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 240-249
```cpp
bool ModuleLinker::shouldLinkFromSource(bool &LinkFromSrc,
                                        const GlobalValue &Dest,
                                        const GlobalValue &Src) {

  // Should we unconditionally use the Src?
  if (shouldOverrideFromSrc()) {
    LinkFromSrc = true;
    return false;
  }

```
- **EN**: Implements logic around `shouldLinkFromSource`, `shouldOverrideFromSrc`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `shouldLinkFromSource`, `shouldOverrideFromSrc` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 250-269
```cpp
  // We always have to add Src if it has appending linkage.
  if (Src.hasAppendingLinkage() || Dest.hasAppendingLinkage()) {
    LinkFromSrc = true;
    return false;
  }

  bool SrcIsDeclaration = Src.isDeclarationForLinker();
  bool DestIsDeclaration = Dest.isDeclarationForLinker();

  if (SrcIsDeclaration) {
    // If Src is external or if both Src & Dest are external..  Just link the
    // external globals, we aren't adding anything.
    if (Src.hasDLLImportStorageClass()) {
      // If one of GVs is marked as DLLImport, result should be dllimport'ed.
      LinkFromSrc = DestIsDeclaration;
      return false;
    }
    // If the Dest is weak, use the source linkage.
    if (Dest.hasExternalWeakLinkage()) {
      LinkFromSrc = true;
```
- **EN**: Implements logic around `hasAppendingLinkage`, `isDeclarationForLinker`, `hasDLLImportStorageClass`, `hasExternalWeakLinkage`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `hasAppendingLinkage`, `isDeclarationForLinker`, `hasDLLImportStorageClass`, `hasExternalWeakLinkage` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 270-282
```cpp
      return false;
    }
    // Link an available_externally over a declaration.
    LinkFromSrc = !Src.isDeclaration() && Dest.isDeclaration();
    return false;
  }

  if (DestIsDeclaration) {
    // If Dest is external but Src is not:
    LinkFromSrc = true;
    return false;
  }

```
- **EN**: Implements logic around `isDeclaration`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `isDeclaration` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 283-293
```cpp
  if (Src.hasCommonLinkage()) {
    if (Dest.hasLinkOnceLinkage() || Dest.hasWeakLinkage()) {
      LinkFromSrc = true;
      return false;
    }

    if (!Dest.hasCommonLinkage()) {
      LinkFromSrc = false;
      return false;
    }

```
- **EN**: Implements logic around `hasCommonLinkage`, `hasLinkOnceLinkage`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `hasCommonLinkage`, `hasLinkOnceLinkage` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 294-306
```cpp
    const DataLayout &DL = Dest.getDataLayout();
    // Functions and aliases may not have common linkage, so both must be
    // GlobalVariables here
    uint64_t DestSize = cast<GlobalVariable>(Dest).getGlobalSize(DL);
    uint64_t SrcSize = cast<GlobalVariable>(Src).getGlobalSize(DL);
    LinkFromSrc = SrcSize > DestSize;
    return false;
  }

  if (Src.isWeakForLinker()) {
    assert(!Dest.hasExternalWeakLinkage());
    assert(!Dest.hasAvailableExternallyLinkage());

```
- **EN**: Implements logic around `getDataLayout`, `cast`, `isWeakForLinker`, `assert`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `getDataLayout`, `cast`, `isWeakForLinker`, `assert` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 307-321
```cpp
    if (Dest.hasLinkOnceLinkage() && Src.hasWeakLinkage()) {
      LinkFromSrc = true;
      return false;
    }

    LinkFromSrc = false;
    return false;
  }

  if (Dest.isWeakForLinker()) {
    assert(Src.hasExternalLinkage());
    LinkFromSrc = true;
    return false;
  }

```
- **EN**: Implements logic around `hasLinkOnceLinkage`, `isWeakForLinker`, `assert`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `hasLinkOnceLinkage`, `isWeakForLinker`, `assert` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 322-333
```cpp
  assert(!Src.hasExternalWeakLinkage());
  assert(!Dest.hasExternalWeakLinkage());
  assert(Dest.hasExternalLinkage() && Src.hasExternalLinkage() &&
         "Unexpected linkage type!");
  return emitError("Linking globals named '" + Src.getName() +
                   "': symbol multiply defined!");
}

bool ModuleLinker::linkIfNeeded(GlobalValue &GV,
                                SmallVectorImpl<GlobalValue *> &GVToClone) {
  GlobalValue *DGV = getLinkedToGlobal(&GV);

```
- **EN**: Implements logic around `assert`, `emitError`, `linkIfNeeded`, `getLinkedToGlobal`; this block emits or serializes data to an external representation; coordinates cross-module linking or optimization state.
- **CN**: 围绕 `assert`, `emitError`, `linkIfNeeded`, `getLinkedToGlobal` 实现具体逻辑；该代码块把数据输出或序列化为外部表示，并协调跨模块链接或优化状态。

### Lines 334-346
```cpp
  if (shouldLinkOnlyNeeded()) {
    // Always import variables with appending linkage.
    if (!GV.hasAppendingLinkage()) {
      // Don't import globals unless they are referenced by the destination
      // module.
      if (!DGV)
        return false;
      // Don't import globals that are already defined in the destination module
      if (!DGV->isDeclaration())
        return false;
    }
  }

```
- **EN**: Implements logic around `shouldLinkOnlyNeeded`, `hasAppendingLinkage`, `isDeclaration`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `shouldLinkOnlyNeeded`, `hasAppendingLinkage`, `isDeclaration` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 347-362
```cpp
  if (DGV && !GV.hasLocalLinkage() && !GV.hasAppendingLinkage()) {
    auto *DGVar = dyn_cast<GlobalVariable>(DGV);
    auto *SGVar = dyn_cast<GlobalVariable>(&GV);
    if (DGVar && SGVar) {
      if (DGVar->isDeclaration() && SGVar->isDeclaration() &&
          (!DGVar->isConstant() || !SGVar->isConstant())) {
        DGVar->setConstant(false);
        SGVar->setConstant(false);
      }
      if (DGVar->hasCommonLinkage() && SGVar->hasCommonLinkage()) {
        MaybeAlign DAlign = DGVar->getAlign();
        MaybeAlign SAlign = SGVar->getAlign();
        MaybeAlign Align = std::nullopt;
        if (DAlign || SAlign)
          Align = std::max(DAlign.valueOrOne(), SAlign.valueOrOne());

```
- **EN**: Implements logic around `hasLocalLinkage`, `dyn_cast`, `isDeclaration`, `isConstant`, and 4 more symbols; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `hasLocalLinkage`, `dyn_cast`, `isDeclaration`, `isConstant`, and 4 more symbols 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 363-372
```cpp
        SGVar->setAlignment(Align);
        DGVar->setAlignment(Align);
      }
    }

    GlobalValue::VisibilityTypes Visibility =
        getMinVisibility(DGV->getVisibility(), GV.getVisibility());
    DGV->setVisibility(Visibility);
    GV.setVisibility(Visibility);

```
- **EN**: Implements logic around `setAlignment`, `getMinVisibility`, `setVisibility`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `setAlignment`, `getMinVisibility`, `setVisibility` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 373-383
```cpp
    GlobalValue::UnnamedAddr UnnamedAddr = GlobalValue::getMinUnnamedAddr(
        DGV->getUnnamedAddr(), GV.getUnnamedAddr());
    DGV->setUnnamedAddr(UnnamedAddr);
    GV.setUnnamedAddr(UnnamedAddr);
  }

  if (!DGV && !shouldOverrideFromSrc() &&
      (GV.hasLocalLinkage() || GV.hasLinkOnceLinkage() ||
       GV.hasAvailableExternallyLinkage()))
    return false;

```
- **EN**: Implements logic around `getMinUnnamedAddr`, `getUnnamedAddr`, `setUnnamedAddr`, `shouldOverrideFromSrc`, and 2 more symbols; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `getMinUnnamedAddr`, `getUnnamedAddr`, `setUnnamedAddr`, `shouldOverrideFromSrc`, and 2 more symbols 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 384-393
```cpp
  if (GV.isDeclaration())
    return false;

  LinkFrom ComdatFrom = LinkFrom::Dst;
  if (const Comdat *SC = GV.getComdat()) {
    std::tie(std::ignore, ComdatFrom) = ComdatsChosen[SC];
    if (ComdatFrom == LinkFrom::Dst)
      return false;
  }

```
- **EN**: Implements logic around `isDeclaration`, `getComdat`, `tie`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `isDeclaration`, `getComdat`, `tie` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 394-403
```cpp
  bool LinkFromSrc = true;
  if (DGV && shouldLinkFromSource(LinkFromSrc, *DGV, GV))
    return true;
  if (DGV && ComdatFrom == LinkFrom::Both)
    GVToClone.push_back(LinkFromSrc ? DGV : &GV);
  if (LinkFromSrc)
    ValuesToLink.insert(&GV);
  return false;
}

```
- **EN**: Implements logic around `shouldLinkFromSource`, `push_back`, `insert`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `shouldLinkFromSource`, `push_back`, `insert` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 404-413
```cpp
void ModuleLinker::addLazyFor(GlobalValue &GV, const IRMover::ValueAdder &Add) {
  // Add these to the internalize list
  if (!GV.hasLinkOnceLinkage() && !GV.hasAvailableExternallyLinkage() &&
      !shouldLinkOnlyNeeded())
    return;

  if (InternalizeCallback)
    Internalize.insert(GV.getName());
  Add(GV);

```
- **EN**: Implements logic around `addLazyFor`, `hasLinkOnceLinkage`, `shouldLinkOnlyNeeded`, `insert`, and 1 more symbols; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `addLazyFor`, `hasLinkOnceLinkage`, `shouldLinkOnlyNeeded`, `insert`, and 1 more symbols 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 414-429
```cpp
  const Comdat *SC = GV.getComdat();
  if (!SC)
    return;
  for (GlobalValue *GV2 : LazyComdatMembers[SC]) {
    GlobalValue *DGV = getLinkedToGlobal(GV2);
    bool LinkFromSrc = true;
    if (DGV && shouldLinkFromSource(LinkFromSrc, *DGV, *GV2))
      return;
    if (!LinkFromSrc)
      continue;
    if (InternalizeCallback)
      Internalize.insert(GV2->getName());
    Add(*GV2);
  }
}

```
- **EN**: Implements logic around `getComdat`, `getLinkedToGlobal`, `shouldLinkFromSource`, `insert`, and 1 more symbols; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `getComdat`, `getLinkedToGlobal`, `shouldLinkFromSource`, `insert`, and 1 more symbols 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 430-441
```cpp
void ModuleLinker::dropReplacedComdat(
    GlobalValue &GV, const DenseSet<const Comdat *> &ReplacedDstComdats) {
  Comdat *C = GV.getComdat();
  if (!C)
    return;
  if (!ReplacedDstComdats.count(C))
    return;
  if (GV.use_empty()) {
    GV.eraseFromParent();
    return;
  }

```
- **EN**: Implements logic around `dropReplacedComdat`, `getComdat`, `count`, `use_empty`, and 1 more symbols; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `dropReplacedComdat`, `getComdat`, `count`, `use_empty`, and 1 more symbols 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 442-461
```cpp
  if (auto *F = dyn_cast<Function>(&GV)) {
    F->deleteBody();
  } else if (auto *Var = dyn_cast<GlobalVariable>(&GV)) {
    Var->setInitializer(nullptr);
  } else {
    auto &Alias = cast<GlobalAlias>(GV);
    Module &M = *Alias.getParent();
    GlobalValue *Declaration;
    if (auto *FTy = dyn_cast<FunctionType>(Alias.getValueType())) {
      Declaration = Function::Create(FTy, GlobalValue::ExternalLinkage, "", &M);
    } else {
      Declaration =
          new GlobalVariable(M, Alias.getValueType(), /*isConstant*/ false,
                             GlobalValue::ExternalLinkage,
                             /*Initializer*/ nullptr);
    }
    Declaration->takeName(&Alias);
    Alias.replaceAllUsesWith(Declaration);
    Alias.eraseFromParent();
  }
```
- **EN**: Implements logic around `dyn_cast`, `deleteBody`, `setInitializer`, `cast`, and 6 more symbols; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `dyn_cast`, `deleteBody`, `setInitializer`, `cast`, and 6 more symbols 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 462-478
```cpp
}

bool ModuleLinker::run() {
  Module &DstM = Mover.getModule();
  DenseSet<const Comdat *> ReplacedDstComdats;
  DenseSet<const Comdat *> NonPrevailingComdats;

  for (const auto &SMEC : SrcM->getComdatSymbolTable()) {
    const Comdat &C = SMEC.getValue();
    if (ComdatsChosen.count(&C))
      continue;
    Comdat::SelectionKind SK;
    LinkFrom From;
    if (getComdatResult(&C, SK, From))
      return true;
    ComdatsChosen[&C] = std::make_pair(SK, From);

```
- **EN**: Implements logic around `run`, `getModule`, `getComdatSymbolTable`, `getValue`, and 3 more symbols; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `run`, `getModule`, `getComdatSymbolTable`, `getValue`, and 3 more symbols 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 479-489
```cpp
    if (From == LinkFrom::Dst)
      NonPrevailingComdats.insert(&C);

    if (From != LinkFrom::Src)
      continue;

    Module::ComdatSymTabType &ComdatSymTab = DstM.getComdatSymbolTable();
    Module::ComdatSymTabType::iterator DstCI = ComdatSymTab.find(C.getName());
    if (DstCI == ComdatSymTab.end())
      continue;

```
- **EN**: Implements logic around `insert`, `getComdatSymbolTable`, `find`, `end`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `insert`, `getComdatSymbolTable`, `find`, `end` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 490-499
```cpp
    // The source comdat is replacing the dest one.
    const Comdat *DstC = &DstCI->second;
    ReplacedDstComdats.insert(DstC);
  }

  // Alias have to go first, since we are not able to find their comdats
  // otherwise.
  for (GlobalAlias &GV : llvm::make_early_inc_range(DstM.aliases()))
    dropReplacedComdat(GV, ReplacedDstComdats);

```
- **EN**: Implements logic around `insert`, `make_early_inc_range`, `dropReplacedComdat`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `insert`, `make_early_inc_range`, `dropReplacedComdat` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 500-519
```cpp
  for (GlobalVariable &GV : llvm::make_early_inc_range(DstM.globals()))
    dropReplacedComdat(GV, ReplacedDstComdats);

  for (Function &GV : llvm::make_early_inc_range(DstM))
    dropReplacedComdat(GV, ReplacedDstComdats);

  if (!NonPrevailingComdats.empty()) {
    DenseSet<GlobalObject *> AliasedGlobals;
    for (auto &GA : SrcM->aliases())
      if (GlobalObject *GO = GA.getAliaseeObject(); GO && GO->getComdat())
        AliasedGlobals.insert(GO);
    for (const Comdat *C : NonPrevailingComdats) {
      SmallVector<GlobalObject *> ToUpdate;
      for (GlobalObject *GO : C->getUsers())
        if (GO->hasPrivateLinkage() && !AliasedGlobals.contains(GO))
          ToUpdate.push_back(GO);
      for (GlobalObject *GO : ToUpdate) {
        GO->setLinkage(GlobalValue::AvailableExternallyLinkage);
        GO->setComdat(nullptr);
      }
```
- **EN**: Implements logic around `make_early_inc_range`, `dropReplacedComdat`, `empty`, `aliases`, and 7 more symbols; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `make_early_inc_range`, `dropReplacedComdat`, `empty`, `aliases`, and 7 more symbols 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 520-532
```cpp
    }
  }

  for (GlobalVariable &GV : SrcM->globals())
    if (GV.hasLinkOnceLinkage())
      if (const Comdat *SC = GV.getComdat())
        LazyComdatMembers[SC].push_back(&GV);

  for (Function &SF : *SrcM)
    if (SF.hasLinkOnceLinkage())
      if (const Comdat *SC = SF.getComdat())
        LazyComdatMembers[SC].push_back(&SF);

```
- **EN**: Implements logic around `globals`, `hasLinkOnceLinkage`, `getComdat`, `push_back`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `globals`, `hasLinkOnceLinkage`, `getComdat`, `push_back` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 533-544
```cpp
  for (GlobalAlias &GA : SrcM->aliases())
    if (GA.hasLinkOnceLinkage())
      if (const Comdat *SC = GA.getComdat())
        LazyComdatMembers[SC].push_back(&GA);

  // Insert all of the globals in src into the DstM module... without linking
  // initializers (which could refer to functions not yet mapped over).
  SmallVector<GlobalValue *, 0> GVToClone;
  for (GlobalVariable &GV : SrcM->globals())
    if (linkIfNeeded(GV, GVToClone))
      return true;

```
- **EN**: Implements logic around `aliases`, `hasLinkOnceLinkage`, `getComdat`, `push_back`, and 2 more symbols; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `aliases`, `hasLinkOnceLinkage`, `getComdat`, `push_back`, and 2 more symbols 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 545-556
```cpp
  for (Function &SF : *SrcM)
    if (linkIfNeeded(SF, GVToClone))
      return true;

  for (GlobalAlias &GA : SrcM->aliases())
    if (linkIfNeeded(GA, GVToClone))
      return true;

  for (GlobalIFunc &GI : SrcM->ifuncs())
    if (linkIfNeeded(GI, GVToClone))
      return true;

```
- **EN**: Implements logic around `linkIfNeeded`, `aliases`, `ifuncs`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `linkIfNeeded`, `aliases`, `ifuncs` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 557-576
```cpp
  // For a variable in a comdat nodeduplicate, its initializer should be
  // preserved (its content may be implicitly used by other members) even if
  // symbol resolution does not pick it. Clone it into an unnamed private
  // variable.
  for (GlobalValue *GV : GVToClone) {
    if (auto *Var = dyn_cast<GlobalVariable>(GV)) {
      auto *NewVar = new GlobalVariable(*Var->getParent(), Var->getValueType(),
                                        Var->isConstant(), Var->getLinkage(),
                                        Var->getInitializer());
      NewVar->copyAttributesFrom(Var);
      NewVar->setVisibility(GlobalValue::DefaultVisibility);
      NewVar->setLinkage(GlobalValue::PrivateLinkage);
      NewVar->setDSOLocal(true);
      NewVar->setComdat(Var->getComdat());
      if (Var->getParent() != &Mover.getModule())
        ValuesToLink.insert(NewVar);
    } else {
      emitError("linking '" + GV->getName() +
                "': non-variables in comdat nodeduplicate are not handled");
    }
```
- **EN**: Implements logic around `dyn_cast`, `GlobalVariable`, `isConstant`, `getInitializer`, and 8 more symbols; this block emits or serializes data to an external representation; coordinates cross-module linking or optimization state.
- **CN**: 围绕 `dyn_cast`, `GlobalVariable`, `isConstant`, `getInitializer`, and 8 more symbols 实现具体逻辑；该代码块把数据输出或序列化为外部表示，并协调跨模块链接或优化状态。

### Lines 577-593
```cpp
  }

  for (unsigned I = 0; I < ValuesToLink.size(); ++I) {
    GlobalValue *GV = ValuesToLink[I];
    const Comdat *SC = GV->getComdat();
    if (!SC)
      continue;
    for (GlobalValue *GV2 : LazyComdatMembers[SC]) {
      GlobalValue *DGV = getLinkedToGlobal(GV2);
      bool LinkFromSrc = true;
      if (DGV && shouldLinkFromSource(LinkFromSrc, *DGV, *GV2))
        return true;
      if (LinkFromSrc)
        ValuesToLink.insert(GV2);
    }
  }

```
- **EN**: Implements logic around `size`, `getComdat`, `getLinkedToGlobal`, `shouldLinkFromSource`, and 1 more symbols; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `size`, `getComdat`, `getLinkedToGlobal`, `shouldLinkFromSource`, and 1 more symbols 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 594-613
```cpp
  if (InternalizeCallback) {
    for (GlobalValue *GV : ValuesToLink)
      Internalize.insert(GV->getName());
  }

  // FIXME: Propagate Errors through to the caller instead of emitting
  // diagnostics.
  bool HasErrors = false;
  if (Error E =
          Mover.move(std::move(SrcM), ValuesToLink.getArrayRef(),
                     IRMover::LazyCallback(
                         [this](GlobalValue &GV, IRMover::ValueAdder Add) {
                           addLazyFor(GV, Add);
                         }),
                     /* IsPerformingImport */ false)) {
    handleAllErrors(std::move(E), [&](ErrorInfoBase &EIB) {
      DstM.getContext().diagnose(LinkDiagnosticInfo(DS_Error, EIB.message()));
      HasErrors = true;
    });
  }
```
- **EN**: Implements logic around `insert`, `move`, `LazyCallback`, `addLazyFor`, and 2 more symbols; this block propagates recoverable errors through LLVM error utilities; emits or serializes data to an external representation; coordinates cross-module linking or optimization state.
- **CN**: 围绕 `insert`, `move`, `LazyCallback`, `addLazyFor`, and 2 more symbols 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并把数据输出或序列化为外部表示，并协调跨模块链接或优化状态。

### Lines 614-624
```cpp
  if (HasErrors)
    return true;

  if (InternalizeCallback)
    InternalizeCallback(DstM, Internalize);

  return false;
}

Linker::Linker(Module &M) : Mover(M) {}

```
- **EN**: Implements logic around `InternalizeCallback`, `Linker`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `InternalizeCallback`, `Linker` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 625-635
```cpp
bool Linker::linkInModule(
    std::unique_ptr<Module> Src, unsigned Flags,
    std::function<void(Module &, const StringSet<> &)> InternalizeCallback) {
  ModuleLinker ModLinker(Mover, std::move(Src), Flags,
                         std::move(InternalizeCallback));
  return ModLinker.run();
}

//===----------------------------------------------------------------------===//
// LinkModules entrypoint.
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `linkInModule`, `function`, `ModLinker`, `move`, and 1 more symbols; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `linkInModule`, `function`, `ModLinker`, `move`, and 1 more symbols 实现具体逻辑；该代码块协调跨模块链接或优化状态。

### Lines 636-648
```cpp

/// This function links two modules together, with the resulting Dest module
/// modified to be the composite of the two input modules. If an error occurs,
/// true is returned and ErrorMsg (if not null) is set to indicate the problem.
/// Upon failure, the Dest module could be in a modified state, and shouldn't be
/// relied on to be consistent.
bool Linker::linkModules(
    Module &Dest, std::unique_ptr<Module> Src, unsigned Flags,
    std::function<void(Module &, const StringSet<> &)> InternalizeCallback) {
  Linker L(Dest);
  return L.linkInModule(std::move(Src), Flags, std::move(InternalizeCallback));
}

```
- **EN**: Implements logic around `linkModules`, `function`, `L`, `linkInModule`; this block propagates recoverable errors through LLVM error utilities; coordinates cross-module linking or optimization state.
- **CN**: 围绕 `linkModules`, `function`, `L`, `linkInModule` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并协调跨模块链接或优化状态。

### Lines 649-657
```cpp
//===----------------------------------------------------------------------===//
// C API.
//===----------------------------------------------------------------------===//

LLVMBool LLVMLinkModules2(LLVMModuleRef Dest, LLVMModuleRef Src) {
  Module *D = unwrap(Dest);
  std::unique_ptr<Module> M(unwrap(Src));
  return Linker::linkModules(*D, std::move(M));
}
```
- **EN**: Implements logic around `unwrap`, `M`, `linkModules`; this block coordinates cross-module linking or optimization state.
- **CN**: 围绕 `unwrap`, `M`, `linkModules` 实现具体逻辑；该代码块协调跨模块链接或优化状态。

## Key Concepts / 关键概念

- **IR linking / IR 链接**:
  - **EN**: Combines modules while reconciling symbols, types, and metadata.
  - **CN**: 在协调符号、类型与元数据的同时合并多个模块。
- **Structured error handling / 结构化错误处理**:
  - **EN**: Uses `Expected`, `Error`, or related helpers to make failures explicit.
  - **CN**: 使用 `Expected`、`Error` 或相关辅助工具显式表示失败。
- **LLVM container usage / LLVM 容器使用**:
  - **EN**: Relies on LLVM ADT containers for performance-conscious in-memory data management.
  - **CN**: 依赖 LLVM ADT 容器来进行注重性能的内存数据管理。

## Dependencies / 依赖关系

- **Direct LLVM/local includes / 直接的 LLVM/本地包含**: `LinkDiagnosticInfo.h`, `llvm-c/Linker.h`, `llvm/ADT/SetVector.h`, `llvm/IR/Comdat.h`, `llvm/IR/GlobalValue.h`, `llvm/IR/LLVMContext.h`, `llvm/IR/Module.h`, `llvm/Linker/Linker.h`, `llvm/Support/Error.h`
- **Subsystem categories / 子系统类别**: LLVM IR core abstractions / LLVM IR 核心抽象 (4), C API declarations / C API 声明 (1), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1), support-library helpers / Support 库辅助功能 (1)
