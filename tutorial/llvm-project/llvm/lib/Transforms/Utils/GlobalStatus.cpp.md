# GlobalStatus.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/GlobalStatus.cpp` | `llvm/lib/Transforms/Utils/GlobalStatus.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements compute status info for globals within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 GlobalStatus 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
//===-- GlobalStatus.cpp - Compute status info for globals -----------------==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/GlobalStatus.h"
#include "llvm/ADT/SmallPtrSet.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/Constant.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/GlobalValue.h"
#include "llvm/IR/GlobalVariable.h"
#include "llvm/IR/InstrTypes.h"
#include "llvm/IR/Instruction.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/IntrinsicInst.h"
#include "llvm/IR/Use.h"
```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。

### Lines 21-39

```cpp
#include "llvm/IR/User.h"
#include "llvm/IR/Value.h"
#include "llvm/Support/AtomicOrdering.h"
#include "llvm/Support/Casting.h"
#include <algorithm>
#include <cassert>

using namespace llvm;

/// Return the stronger of the two ordering. If the two orderings are acquire
/// and release, then return AcquireRelease.
///
static AtomicOrdering strongerOrdering(AtomicOrdering X, AtomicOrdering Y) {
  if ((X == AtomicOrdering::Acquire && Y == AtomicOrdering::Release) ||
      (Y == AtomicOrdering::Acquire && X == AtomicOrdering::Release))
    return AtomicOrdering::AcquireRelease;
  return (AtomicOrdering)std::max((unsigned)X, (unsigned)Y);
}

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include strongerOrdering, which carry the local state, declarations, or transformation entry points for this region.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 strongerOrdering，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 40-63

```cpp
/// It is safe to destroy a constant iff it is only used by constants itself.
/// Note that while constants cannot be cyclic, they can be tree-like, so we
/// should keep a visited set to avoid exponential runtime.
bool llvm::isSafeToDestroyConstant(const Constant *C) {
  SmallVector<const Constant *, 8> Worklist;
  SmallPtrSet<const Constant *, 8> Visited;
  Worklist.push_back(C);
  while (!Worklist.empty()) {
    const Constant *C = Worklist.pop_back_val();
    if (!Visited.insert(C).second)
      continue;
    if (isa<GlobalValue>(C) || isa<ConstantData>(C))
      return false;

    for (const User *U : C->users()) {
      if (const Constant *CU = dyn_cast<Constant>(U))
        Worklist.push_back(CU);
      else
        return false;
    }
  }
  return true;
}

```
- EN: Core entities appearing here include isSafeToDestroyConstant, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 isSafeToDestroyConstant，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 64-83

```cpp
static bool analyzeGlobalAux(const Value *V, GlobalStatus &GS,
                             SmallPtrSetImpl<const Value *> &VisitedUsers) {
  if (const GlobalVariable *GV = dyn_cast<GlobalVariable>(V))
    if (GV->isExternallyInitialized())
      GS.StoredType = GlobalStatus::StoredOnce;

  for (const Use &U : V->uses()) {
    const User *UR = U.getUser();
    if (const Constant *C = dyn_cast<Constant>(UR)) {
      const ConstantExpr *CE = dyn_cast<ConstantExpr>(C);
      if (CE && isa<PointerType>(CE->getType())) {
        // Recursively analyze pointer-typed constant expressions.
        // FIXME: Do we need to add constexpr selects to VisitedUsers?
        if (analyzeGlobalAux(CE, GS, VisitedUsers))
          return true;
      } else {
        // Ignore dead constant users.
        if (!isSafeToDestroyConstant(C))
          return true;
      }
```
- EN: This region continues the GlobalStatus implementation with local helper logic centered on Value, GlobalStatus, SmallPtrSetImpl, VisitedUsers.
- CN: 这一段延续了 GlobalStatus 的主体实现，围绕 Value, GlobalStatus, SmallPtrSetImpl, VisitedUsers 等局部辅助逻辑展开。

### Lines 84-106

```cpp
    } else if (const Instruction *I = dyn_cast<Instruction>(UR)) {
      if (!GS.HasMultipleAccessingFunctions) {
        const Function *F = I->getParent()->getParent();
        if (!GS.AccessingFunction)
          GS.AccessingFunction = F;
        else if (GS.AccessingFunction != F)
          GS.HasMultipleAccessingFunctions = true;
      }
      if (const LoadInst *LI = dyn_cast<LoadInst>(I)) {
        GS.IsLoaded = true;
        // Don't hack on volatile loads.
        if (LI->isVolatile())
          return true;
        GS.Ordering = strongerOrdering(GS.Ordering, LI->getOrdering());
      } else if (const StoreInst *SI = dyn_cast<StoreInst>(I)) {
        // Don't allow a store OF the address, only stores TO the address.
        if (SI->getOperand(0) == V)
          return true;

        // Don't hack on volatile stores.
        if (SI->isVolatile())
          return true;

```
- EN: This region continues the GlobalStatus implementation with local helper logic centered on Instruction, HasMultipleAccessingFunctions, Function, AccessingFunction.
- CN: 这一段延续了 GlobalStatus 的主体实现，围绕 Instruction, HasMultipleAccessingFunctions, Function, AccessingFunction 等局部辅助逻辑展开。

### Lines 107-125

```cpp
        ++GS.NumStores;

        GS.Ordering = strongerOrdering(GS.Ordering, SI->getOrdering());

        // If this is a direct store to the global (i.e., the global is a scalar
        // value, not an aggregate), keep more specific information about
        // stores.
        if (GS.StoredType != GlobalStatus::Stored) {
          const Value *Ptr = SI->getPointerOperand()->stripPointerCasts();
          if (const GlobalVariable *GV = dyn_cast<GlobalVariable>(Ptr)) {
            Value *StoredVal = SI->getOperand(0);

            if (Constant *C = dyn_cast<Constant>(StoredVal)) {
              if (C->isThreadDependent()) {
                // The stored value changes between threads; don't track it.
                return true;
              }
            }

```
- EN: This region continues the GlobalStatus implementation with local helper logic centered on NumStores, Ordering, StoredType, GlobalStatus.
- CN: 这一段延续了 GlobalStatus 的主体实现，围绕 NumStores, Ordering, StoredType, GlobalStatus 等局部辅助逻辑展开。

### Lines 126-145

```cpp
            if (GV->hasInitializer() && StoredVal == GV->getInitializer()) {
              if (GS.StoredType < GlobalStatus::InitializerStored)
                GS.StoredType = GlobalStatus::InitializerStored;
            } else if (isa<LoadInst>(StoredVal) &&
                       cast<LoadInst>(StoredVal)->getOperand(0) == GV) {
              if (GS.StoredType < GlobalStatus::InitializerStored)
                GS.StoredType = GlobalStatus::InitializerStored;
            } else if (GS.StoredType < GlobalStatus::StoredOnce) {
              GS.StoredType = GlobalStatus::StoredOnce;
              GS.StoredOnceStore = SI;
            } else if (GS.StoredType == GlobalStatus::StoredOnce &&
                       GS.getStoredOnceValue() == StoredVal) {
              // noop.
            } else {
              GS.StoredType = GlobalStatus::Stored;
            }
          } else {
            GS.StoredType = GlobalStatus::Stored;
          }
        }
```
- EN: This region continues the GlobalStatus implementation with local helper logic centered on StoredVal, StoredType, GlobalStatus, InitializerStored.
- CN: 这一段延续了 GlobalStatus 的主体实现，围绕 StoredVal, StoredType, GlobalStatus, InitializerStored 等局部辅助逻辑展开。

### Lines 146-165

```cpp
      } else if (isa<GetElementPtrInst>(I) || isa<AddrSpaceCastInst>(I)) {
        // Skip over GEPs; we don't care about the type or offset
        // of the pointer.
        if (analyzeGlobalAux(I, GS, VisitedUsers))
          return true;
      } else if (isa<SelectInst>(I) || isa<PHINode>(I)) {
        // Look through selects and PHIs to find if the pointer is
        // conditionally accessed. Make sure we only visit an instruction
        // once; otherwise, we can get infinite recursion or exponential
        // compile time.
        if (VisitedUsers.insert(I).second)
          if (analyzeGlobalAux(I, GS, VisitedUsers))
            return true;
      } else if (isa<CmpInst>(I)) {
        GS.IsCompared = true;
      } else if (const MemTransferInst *MTI = dyn_cast<MemTransferInst>(I)) {
        if (MTI->isVolatile())
          return true;
        if (MTI->getArgOperand(0) == V)
          GS.StoredType = GlobalStatus::Stored;
```
- EN: This region continues the GlobalStatus implementation with local helper logic centered on GetElementPtrInst, AddrSpaceCastInst, Skip, GEPs.
- CN: 这一段延续了 GlobalStatus 的主体实现，围绕 GetElementPtrInst, AddrSpaceCastInst, Skip, GEPs 等局部辅助逻辑展开。

### Lines 166-185

```cpp
        if (MTI->getArgOperand(1) == V)
          GS.IsLoaded = true;
      } else if (const MemSetInst *MSI = dyn_cast<MemSetInst>(I)) {
        assert(MSI->getArgOperand(0) == V && "Memset only takes one pointer!");
        if (MSI->isVolatile())
          return true;
        GS.StoredType = GlobalStatus::Stored;
      } else if (const auto *CB = dyn_cast<CallBase>(I)) {
        if (CB->getIntrinsicID() == Intrinsic::threadlocal_address) {
          if (analyzeGlobalAux(I, GS, VisitedUsers))
            return true;
        } else {
          if (!CB->isCallee(&U))
            return true;
          GS.IsLoaded = true;
        }
      } else {
        return true; // Any other non-load instruction might take address!
      }
    } else {
```
- EN: This region continues the GlobalStatus implementation with local helper logic centered on MTI, IsLoaded, MemSetInst, MSI.
- CN: 这一段延续了 GlobalStatus 的主体实现，围绕 MTI, IsLoaded, MemSetInst, MSI 等局部辅助逻辑展开。

### Lines 186-199

```cpp
      // Otherwise must be some other user.
      return true;
    }
  }

  return false;
}

GlobalStatus::GlobalStatus() = default;

bool GlobalStatus::analyzeGlobal(const Value *V, GlobalStatus &GS) {
  SmallPtrSet<const Value *, 16> VisitedUsers;
  return analyzeGlobalAux(V, GS, VisitedUsers);
}
```
- EN: Core entities appearing here include analyzeGlobal, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 analyzeGlobal，它们承载了该区间涉及的局部状态、声明或变换入口。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `strongerOrdering, isSafeToDestroyConstant, analyzeGlobal` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`strongerOrdering, isSafeToDestroyConstant, analyzeGlobal` 构成该文件对外 API 与主要实现挂钩。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/Constant.h`, `llvm/IR/Constants.h`, `llvm/IR/GlobalValue.h`, `llvm/IR/GlobalVariable.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/Use.h`, `llvm/IR/User.h`, `llvm/IR/Value.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/Constant.h`, `llvm/IR/Constants.h`, `llvm/IR/GlobalValue.h`, `llvm/IR/GlobalVariable.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/Use.h`, `llvm/IR/User.h`, `llvm/IR/Value.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/AtomicOrdering.h`, `llvm/Support/Casting.h`, `llvm/ADT/SmallPtrSet.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/AtomicOrdering.h`, `llvm/Support/Casting.h`, `llvm/ADT/SmallPtrSet.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `algorithm`, `cassert` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`algorithm`, `cassert` 为 LLVM API 之外的 C++ 语言工具提供支持。
