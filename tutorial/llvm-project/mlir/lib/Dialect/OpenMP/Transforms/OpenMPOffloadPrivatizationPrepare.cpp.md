# OpenMPOffloadPrivatizationPrepare.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/OpenMP/Transforms/OpenMPOffloadPrivatizationPrepare.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements OpenMP dialect support for rewrite patterns, passes, and IR-to-IR transformation logic, centered on `OpenMPOffloadPrivatizationPrepare`.
  - **CN**: 实现 OpenMP 方言中围绕 `OpenMPOffloadPrivatizationPrepare` 的重写模式、Pass 以及 IR 到 IR 的变换逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
//===- OpenMPOffloadPrivatizationPrepare.cpp - Prepare OMP privatization --===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Analysis/SliceAnalysis.h"
#include "mlir/Dialect/LLVMIR/FunctionCallUtils.h"
#include "mlir/Dialect/LLVMIR/LLVMDialect.h"
#include "mlir/Dialect/OpenMP/OpenMPDialect.h"
#include "mlir/IR/Builders.h"
#include "mlir/IR/Dominance.h"
#include "mlir/IR/IRMapping.h"
#include "mlir/Pass/Pass.h"
#include "mlir/Support/LLVM.h"
#include "llvm/Support/DebugLog.h"
#include "llvm/Support/FormatVariadic.h"
#include <cstdint>
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Analysis/SliceAnalysis.h`, `mlir/Dialect/LLVMIR/FunctionCallUtils.h`, `mlir/Dialect/LLVMIR/LLVMDialect.h`, `mlir/Dialect/OpenMP/OpenMPDialect.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Analysis/SliceAnalysis.h`, `mlir/Dialect/LLVMIR/FunctionCallUtils.h`, `mlir/Dialect/LLVMIR/LLVMDialect.h`, `mlir/Dialect/OpenMP/OpenMPDialect.h`。

### Lines 21-31
```cpp
#include <iterator>
#include <utility>

//===----------------------------------------------------------------------===//
// A pass that prepares OpenMP code for translation of delayed privatization
// in the context of deferred target tasks. Deferred target tasks are created
// when the nowait clause is used on the target directive.
//===----------------------------------------------------------------------===//

#define DEBUG_TYPE "omp-prepare-for-offload-privatization"

```
- **EN**: Pulls in the headers needed by this translation unit, including `iterator`, `utility`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `iterator`, `utility`。

### Lines 32-42
```cpp
namespace mlir {
namespace omp {

#define GEN_PASS_DEF_PREPAREFOROMPOFFLOADPRIVATIZATIONPASS
#include "mlir/Dialect/OpenMP/Transforms/Passes.h.inc"

} // namespace omp
} // namespace mlir

using namespace mlir;
namespace {
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/OpenMP/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/OpenMP/Transforms/Passes.h.inc`。

### Lines 43-54
```cpp

//===----------------------------------------------------------------------===//
// PrepareForOMPOffloadPrivatizationPass
//===----------------------------------------------------------------------===//

class PrepareForOMPOffloadPrivatizationPass
    : public omp::impl::PrepareForOMPOffloadPrivatizationPassBase<
          PrepareForOMPOffloadPrivatizationPass> {

  void runOnOperation() override {
    ModuleOp mod = getOperation();

```
- **EN**: Introduces declarations for `PrepareForOMPOffloadPrivatizationPass`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `PrepareForOMPOffloadPrivatizationPass` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 55-72
```cpp
    // In this pass, we make host-allocated privatized variables persist for
    // deferred target tasks by copying them to the heap. Once the target task
    // is done, this heap memory is freed. Since all of this happens on the host
    // we can skip device modules.
    auto offloadModuleInterface =
        dyn_cast<omp::OffloadModuleInterface>(mod.getOperation());
    if (offloadModuleInterface && offloadModuleInterface.getIsTargetDevice())
      return;

    getOperation()->walk([&](omp::TargetOp targetOp) {
      if (!hasPrivateVars(targetOp) || !isTargetTaskDeferred(targetOp))
        return;
      IRRewriter rewriter(&getContext());
      OperandRange privateVars = targetOp.getPrivateVars();
      SmallVector<mlir::Value> newPrivVars;
      Value fakeDependVar;
      omp::TaskOp cleanupTaskOp;

```
- **EN**: Implements logic around `OffloadModuleInterface>`, `getIsTargetDevice`, `getOperation`, `hasPrivateVars`, and 2 more symbols; this block packages logic as an MLIR pass or pass helper; uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `OffloadModuleInterface>`, `getIsTargetDevice`, `getOperation`, `hasPrivateVars`, and 2 more symbols 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件，并使用重写模式基础设施变换操作。

### Lines 73-87
```cpp
      newPrivVars.reserve(privateVars.size());
      std::optional<ArrayAttr> privateSyms = targetOp.getPrivateSyms();
      for (auto [privVarIdx, privVarSymPair] :
           llvm::enumerate(llvm::zip_equal(privateVars, *privateSyms))) {
        Value privVar = std::get<0>(privVarSymPair);
        Attribute privSym = std::get<1>(privVarSymPair);

        omp::PrivateClauseOp privatizer = findPrivatizer(targetOp, privSym);
        if (!privatizer.needsMap()) {
          newPrivVars.push_back(privVar);
          continue;
        }
        bool isFirstPrivate = privatizer.getDataSharingType() ==
                              omp::DataSharingClauseType::FirstPrivate;

```
- **EN**: Implements logic around `reserve`, `getPrivateSyms`, `enumerate`, `get`, and 4 more symbols.
- **CN**: 围绕 `reserve`, `getPrivateSyms`, `enumerate`, `get`, and 4 more symbols 实现具体逻辑。

### Lines 88-107
```cpp
        Value mappedValue = targetOp.getMappedValueForPrivateVar(privVarIdx);
        auto mapInfoOp = cast<omp::MapInfoOp>(mappedValue.getDefiningOp());

        if (mapInfoOp.getMapCaptureType() == omp::VariableCaptureKind::ByCopy) {
          newPrivVars.push_back(privVar);
          continue;
        }

        // For deferred target tasks (!$omp target nowait), we need to keep
        // a copy of the original, i.e. host variable being privatized so
        // that it is available when the target task is eventually executed.
        // We do this by first allocating as much heap memory as is needed by
        // the original variable. Then, we use the init and copy regions of the
        // privatizer, an instance of omp::PrivateClauseOp to set up the heap-
        // allocated copy.
        // After the target task is done, we need to use the dealloc region
        // of the privatizer to clean up everything. We also need to free
        // the heap memory we allocated. But due to the deferred nature
        // of the target task, we cannot simply deallocate right after the
        // omp.target operation else we may end up freeing memory before
```
- **EN**: Implements logic around `getMappedValueForPrivateVar`, `MapInfoOp>`, `getMapCaptureType`, `push_back`.
- **CN**: 围绕 `getMappedValueForPrivateVar`, `MapInfoOp>`, `getMapCaptureType`, `push_back` 实现具体逻辑。

### Lines 108-122
```cpp
        // its eventual use by the target task. So, we create a dummy
        // dependence between the target task and new omp.task. In the omp.task,
        // we do all the cleanup. So, we end up with the following structure
        //
        // omp.target map_entries(..) ... nowait depend(out:fakeDependVar) {
        //   ...
        //   omp.terminator
        // }
        // omp.task depend(in: fakeDependVar) {
        //   /*cleanup_code*/
        //   omp.terminator
        // }
        // fakeDependVar is the address of the first heap-allocated copy of the
        // host variable being privatized.

```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 123-133
```cpp
        bool needsCleanupTask = !privatizer.getDeallocRegion().empty();

        // Allocate heap memory that corresponds to the type of memory
        // pointed to by varPtr
        // For boxchars this won't be a pointer. But, MapsForPrivatizedSymbols
        // should have mapped the pointer to the boxchar so use that as varPtr.
        Value varPtr = mapInfoOp.getVarPtr();
        Type varType = mapInfoOp.getVarPtrType();
        bool isPrivatizedByValue =
            !isa<LLVM::LLVMPointerType>(privVar.getType());

```
- **EN**: Implements logic around `getDeallocRegion`, `getVarPtr`, `getVarPtrType`, `LLVMPointerType>`.
- **CN**: 围绕 `getDeallocRegion`, `getVarPtr`, `getVarPtrType`, `LLVMPointerType>` 实现具体逻辑。

### Lines 134-145
```cpp
        assert(isa<LLVM::LLVMPointerType>(varPtr.getType()));
        Value heapMem =
            allocateHeapMem(targetOp, varPtr, varType, mod, rewriter);
        if (!heapMem)
          targetOp.emitError(
              "Unable to allocate heap memory when trying to move "
              "a private variable out of the stack and into the "
              "heap for use by a deferred target task");

        if (needsCleanupTask && !fakeDependVar)
          fakeDependVar = heapMem;

```
- **EN**: Implements logic around `assert`, `allocateHeapMem`, `emitError`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `assert`, `allocateHeapMem`, `emitError` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 146-156
```cpp
        // The types of private vars should match before and after the
        // transformation. In particular, if the type is a pointer,
        // simply record the newly allocated malloc location as the
        // new private variable. If, however, the type is not a pointer
        // then, we need to load the value from the newly allocated
        // location. We'll insert that load later after we have updated
        // the malloc'd location with the contents of the original
        // variable.
        if (!isPrivatizedByValue)
          newPrivVars.push_back(heapMem);

```
- **EN**: Implements logic around `push_back`.
- **CN**: 围绕 `push_back` 实现具体逻辑。

### Lines 157-175
```cpp
        // We now need to copy the original private variable into the newly
        // allocated location in the heap.
        // Find the earliest insertion point for the copy. This will be before
        // the first in the list of omp::MapInfoOp instances that use varPtr.
        // After the copy these omp::MapInfoOp instances will refer to heapMem
        // instead.
        Operation *varPtrDefiningOp = varPtr.getDefiningOp();
        DenseSet<Operation *> users;
        if (varPtrDefiningOp) {
          users.insert(varPtrDefiningOp->user_begin(),
                       varPtrDefiningOp->user_end());
        } else {
          auto blockArg = cast<BlockArgument>(varPtr);
          users.insert(blockArg.user_begin(), blockArg.user_end());
        }
        auto usesVarPtr = [&users](Operation *op) -> bool {
          return users.count(op);
        };

```
- **EN**: Implements logic around `getDefiningOp`, `insert`, `user_end`, `count`.
- **CN**: 围绕 `getDefiningOp`, `insert`, `user_end`, `count` 实现具体逻辑。

### Lines 176-189
```cpp
        SmallVector<Operation *> chainOfOps;
        chainOfOps.push_back(mapInfoOp);
        for (auto member : mapInfoOp.getMembers()) {
          omp::MapInfoOp memberMap =
              cast<omp::MapInfoOp>(member.getDefiningOp());
          if (usesVarPtr(memberMap))
            chainOfOps.push_back(memberMap);
          if (memberMap.getVarPtrPtr()) {
            Operation *defOp = memberMap.getVarPtrPtr().getDefiningOp();
            if (defOp && usesVarPtr(defOp))
              chainOfOps.push_back(defOp);
          }
        }

```
- **EN**: Implements logic around `push_back`, `getMembers`, `MapInfoOp>`, `usesVarPtr`, and 1 more symbols.
- **CN**: 围绕 `push_back`, `getMembers`, `MapInfoOp>`, `usesVarPtr`, and 1 more symbols 实现具体逻辑。

### Lines 190-201
```cpp
        DominanceInfo dom;
        llvm::sort(chainOfOps, [&](Operation *l, Operation *r) {
          if (l == r)
            return false;
          return dom.properlyDominates(l, r);
        });

        rewriter.setInsertionPoint(chainOfOps.front());

        Operation *firstOp = chainOfOps.front();
        Location loc = firstOp->getLoc();

```
- **EN**: Implements logic around `sort`, `properlyDominates`, `setInsertionPoint`, `front`, and 1 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `sort`, `properlyDominates`, `setInsertionPoint`, `front`, and 1 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 202-213
```cpp
        // Create a llvm.func for 'region' that is marked always_inline and call
        // it.
        auto createAlwaysInlineFuncAndCallIt =
            [&](Region &region, llvm::StringRef funcName,
                llvm::ArrayRef<Value> args, bool returnsValue) -> Value {
          assert(!region.empty() && "region cannot be empty");
          LLVM::LLVMFuncOp func = createFuncOpForRegion(
              loc, mod, region, funcName, rewriter, returnsValue);
          auto call = LLVM::CallOp::create(rewriter, loc, func, args);
          return call.getResult();
        };

```
- **EN**: Implements logic around `assert`, `createFuncOpForRegion`, `create`, `getResult`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `assert`, `createFuncOpForRegion`, `create`, `getResult` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 214-231
```cpp
        Value moldArg, newArg;
        if (isPrivatizedByValue) {
          moldArg = LLVM::LoadOp::create(rewriter, loc, varType, varPtr);
          newArg = LLVM::LoadOp::create(rewriter, loc, varType, heapMem);
        } else {
          moldArg = varPtr;
          newArg = heapMem;
        }

        Value initializedVal;
        if (!privatizer.getInitRegion().empty())
          initializedVal = createAlwaysInlineFuncAndCallIt(
              privatizer.getInitRegion(),
              llvm::formatv("{0}_{1}", privatizer.getSymName(), "init").str(),
              {moldArg, newArg}, /*returnsValue=*/true);
        else
          initializedVal = newArg;

```
- **EN**: Implements logic around `create`, `getInitRegion`, `createAlwaysInlineFuncAndCallIt`, `formatv`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `create`, `getInitRegion`, `createAlwaysInlineFuncAndCallIt`, `formatv` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 232-251
```cpp
        if (isFirstPrivate && !privatizer.getCopyRegion().empty())
          initializedVal = createAlwaysInlineFuncAndCallIt(
              privatizer.getCopyRegion(),
              llvm::formatv("{0}_{1}", privatizer.getSymName(), "copy").str(),
              {moldArg, initializedVal}, /*returnsValue=*/true);

        if (isPrivatizedByValue)
          (void)LLVM::StoreOp::create(rewriter, loc, initializedVal, heapMem);

        // clone origOp, replace all uses of varPtr with heapMem and
        // erase origOp.
        auto cloneModifyAndErase = [&](Operation *origOp) -> Operation * {
          Operation *clonedOp = rewriter.clone(*origOp);
          rewriter.replaceAllOpUsesWith(origOp, clonedOp);
          rewriter.modifyOpInPlace(clonedOp, [&]() {
            clonedOp->replaceUsesOfWith(varPtr, heapMem);
          });
          rewriter.eraseOp(origOp);
          return clonedOp;
        };
```
- **EN**: Implements logic around `getCopyRegion`, `createAlwaysInlineFuncAndCallIt`, `formatv`, `create`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getCopyRegion`, `createAlwaysInlineFuncAndCallIt`, `formatv`, `create`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 252-268
```cpp

        // Now that we have set up the heap-allocated copy of the private
        // variable, rewrite all the uses of the original variable with
        // the heap-allocated variable.
        rewriter.setInsertionPoint(targetOp);
        mapInfoOp = cast<omp::MapInfoOp>(cloneModifyAndErase(mapInfoOp));
        rewriter.setInsertionPoint(mapInfoOp);

        // Fix any members that may use varPtr to now use heapMem
        for (auto member : mapInfoOp.getMembers()) {
          auto memberMapInfoOp = cast<omp::MapInfoOp>(member.getDefiningOp());
          if (!usesVarPtr(memberMapInfoOp))
            continue;
          memberMapInfoOp =
              cast<omp::MapInfoOp>(cloneModifyAndErase(memberMapInfoOp));
          rewriter.setInsertionPoint(memberMapInfoOp);

```
- **EN**: Implements logic around `setInsertionPoint`, `MapInfoOp>`, `getMembers`, `usesVarPtr`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `setInsertionPoint`, `MapInfoOp>`, `getMembers`, `usesVarPtr` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 269-287
```cpp
          if (memberMapInfoOp.getVarPtrPtr()) {
            Operation *varPtrPtrdefOp =
                memberMapInfoOp.getVarPtrPtr().getDefiningOp();
            rewriter.setInsertionPoint(cloneModifyAndErase(varPtrPtrdefOp));
          }
        }

        // If the type of the private variable is not a pointer,
        // which is typically the case with !fir.boxchar types, then
        // we need to ensure that the new private variable is also
        // not a pointer. Insert a load from heapMem right before
        // targetOp.
        if (isPrivatizedByValue) {
          rewriter.setInsertionPoint(targetOp);
          auto newPrivVar = LLVM::LoadOp::create(rewriter, mapInfoOp.getLoc(),
                                                 varType, heapMem);
          newPrivVars.push_back(newPrivVar);
        }

```
- **EN**: Implements logic around `getVarPtrPtr`, `setInsertionPoint`, `create`, `push_back`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getVarPtrPtr`, `setInsertionPoint`, `create`, `push_back` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 288-307
```cpp
        // Deallocate
        if (needsCleanupTask) {
          if (!cleanupTaskOp) {
            assert(fakeDependVar &&
                   "Need a valid value to set up a dependency");
            rewriter.setInsertionPointAfter(targetOp);
            omp::TaskOperands taskOperands;
            auto inDepend = omp::ClauseTaskDependAttr::get(
                rewriter.getContext(), omp::ClauseTaskDepend::taskdependin);
            taskOperands.dependKinds.push_back(inDepend);
            taskOperands.dependVars.push_back(fakeDependVar);
            cleanupTaskOp = omp::TaskOp::create(rewriter, loc, taskOperands);
            Block *taskBlock = rewriter.createBlock(&cleanupTaskOp.getRegion());
            rewriter.setInsertionPointToEnd(taskBlock);
            omp::TerminatorOp::create(rewriter, cleanupTaskOp.getLoc());
          }
          rewriter.setInsertionPointToStart(
              &*cleanupTaskOp.getRegion().getBlocks().begin());
          (void)createAlwaysInlineFuncAndCallIt(
              privatizer.getDeallocRegion(),
```
- **EN**: Implements logic around `assert`, `setInsertionPointAfter`, `get`, `getContext`, and 8 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `assert`, `setInsertionPointAfter`, `get`, `getContext`, and 8 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 308-327
```cpp
              llvm::formatv("{0}_{1}", privatizer.getSymName(), "dealloc")
                  .str(),
              {initializedVal}, /*returnsValue=*/false);
          llvm::FailureOr<LLVM::LLVMFuncOp> freeFunc =
              LLVM::lookupOrCreateFreeFn(rewriter, mod);
          assert(llvm::succeeded(freeFunc) &&
                 "Could not find free in the module");
          (void)LLVM::CallOp::create(rewriter, loc, freeFunc.value(),
                                     ValueRange{heapMem});
        }
      }
      assert(newPrivVars.size() == privateVars.size() &&
             "The number of private variables must match before and after "
             "transformation");
      if (fakeDependVar) {
        omp::ClauseTaskDependAttr outDepend = omp::ClauseTaskDependAttr::get(
            rewriter.getContext(), omp::ClauseTaskDepend::taskdependout);
        SmallVector<Attribute> newDependKinds;
        if (!targetOp.getDependVars().empty()) {
          std::optional<ArrayAttr> dependKinds = targetOp.getDependKinds();
```
- **EN**: Implements logic around `formatv`, `str`, `lookupOrCreateFreeFn`, `assert`, and 5 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `formatv`, `str`, `lookupOrCreateFreeFn`, `assert`, and 5 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 328-342
```cpp
          assert(dependKinds && "bad depend clause in omp::TargetOp");
          llvm::copy(*dependKinds, std::back_inserter(newDependKinds));
        }
        newDependKinds.push_back(outDepend);
        ArrayAttr newDependKindsAttr =
            ArrayAttr::get(rewriter.getContext(), newDependKinds);
        targetOp.getDependVarsMutable().append(fakeDependVar);
        targetOp.setDependKindsAttr(newDependKindsAttr);
      }
      rewriter.setInsertionPoint(targetOp);
      targetOp.getPrivateVarsMutable().clear();
      targetOp.getPrivateVarsMutable().assign(newPrivVars);
    });
  }

```
- **EN**: Implements logic around `assert`, `copy`, `push_back`, `get`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `assert`, `copy`, `push_back`, `get`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 343-360
```cpp
private:
  bool hasPrivateVars(omp::TargetOp targetOp) const {
    return !targetOp.getPrivateVars().empty();
  }

  bool isTargetTaskDeferred(omp::TargetOp targetOp) const {
    return targetOp.getNowait();
  }

  template <typename OpTy>
  omp::PrivateClauseOp findPrivatizer(OpTy op, Attribute privSym) const {
    SymbolRefAttr privatizerName = llvm::cast<SymbolRefAttr>(privSym);
    omp::PrivateClauseOp privatizer =
        SymbolTable::lookupNearestSymbolFrom<omp::PrivateClauseOp>(
            op, privatizerName);
    return privatizer;
  }

```
- **EN**: Implements logic around `hasPrivateVars`, `getPrivateVars`, `isTargetTaskDeferred`, `getNowait`, and 2 more symbols.
- **CN**: 围绕 `hasPrivateVars`, `getPrivateVars`, `isTargetTaskDeferred`, `getNowait`, and 2 more symbols 实现具体逻辑。

### Lines 361-376
```cpp
  // Get the (compile-time constant) size of varType as per the
  // given DataLayout dl.
  std::int64_t getSizeInBytes(const DataLayout &dl, Type varType) const {
    llvm::TypeSize size = dl.getTypeSize(varType);
    unsigned short alignment = dl.getTypeABIAlignment(varType);
    return llvm::alignTo(size, alignment);
  }

  LLVM::LLVMFuncOp getMalloc(ModuleOp mod, IRRewriter &rewriter) const {
    llvm::FailureOr<LLVM::LLVMFuncOp> mallocCall =
        LLVM::lookupOrCreateMallocFn(rewriter, mod, rewriter.getI64Type());
    assert(llvm::succeeded(mallocCall) &&
           "Could not find malloc in the module");
    return mallocCall.value();
  }

```
- **EN**: Implements logic around `getSizeInBytes`, `getTypeSize`, `getTypeABIAlignment`, `alignTo`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `getSizeInBytes`, `getTypeSize`, `getTypeABIAlignment`, `alignTo`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 377-391
```cpp
  Value allocateHeapMem(omp::TargetOp targetOp, Value privVar, Type varType,
                        ModuleOp mod, IRRewriter &rewriter) const {
    OpBuilder::InsertionGuard guard(rewriter);
    Value varPtr = privVar;
    Operation *definingOp = varPtr.getDefiningOp();
    BlockArgument blockArg;
    if (!definingOp) {
      blockArg = mlir::dyn_cast<BlockArgument>(varPtr);
      rewriter.setInsertionPointToStart(blockArg.getParentBlock());
    } else {
      rewriter.setInsertionPoint(definingOp);
    }
    Location loc = definingOp ? definingOp->getLoc() : blockArg.getLoc();
    LLVM::LLVMFuncOp mallocFn = getMalloc(mod, rewriter);

```
- **EN**: Implements logic around `allocateHeapMem`, `guard`, `getDefiningOp`, `setInsertionPointToStart`, and 3 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `allocateHeapMem`, `guard`, `getDefiningOp`, `setInsertionPointToStart`, and 3 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 392-405
```cpp
    assert(mod.getDataLayoutSpec() &&
           "MLIR module with no datalayout spec not handled yet");

    const DataLayout &dl = DataLayout(mod);
    std::int64_t distance = getSizeInBytes(dl, varType);

    Value sizeBytes = LLVM::ConstantOp::create(
        rewriter, loc, mallocFn.getFunctionType().getParamType(0), distance);

    auto mallocCallOp =
        LLVM::CallOp::create(rewriter, loc, mallocFn, ValueRange{sizeBytes});
    return mallocCallOp.getResult();
  }

```
- **EN**: Implements logic around `assert`, `DataLayout`, `getSizeInBytes`, `create`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `assert`, `DataLayout`, `getSizeInBytes`, `create`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 406-416
```cpp
  // Create a function for srcRegion and attribute it to be always_inline.
  // The big assumption here is that srcRegion is one of init, copy or dealloc
  // regions of a omp::PrivateClauseop. Accordingly, the return type is assumed
  // to either be the same as the types of the two arguments of the region (for
  // init and copy regions) or void as would be the case for dealloc regions.
  LLVM::LLVMFuncOp createFuncOpForRegion(Location loc, ModuleOp mod,
                                         Region &srcRegion,
                                         llvm::StringRef funcName,
                                         IRRewriter &rewriter,
                                         bool returnsValue = false) {

```
- **EN**: Implements logic around `createFuncOpForRegion`; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `createFuncOpForRegion` 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 417-430
```cpp
    OpBuilder::InsertionGuard guard(rewriter);
    rewriter.setInsertionPoint(mod.getBody(), mod.getBody()->end());
    Region clonedRegion;
    IRMapping mapper;
    srcRegion.cloneInto(&clonedRegion, mapper);

    SmallVector<Type> paramTypes;
    llvm::copy(srcRegion.getArgumentTypes(), std::back_inserter(paramTypes));
    Type resultType = returnsValue
                          ? srcRegion.getArgument(0).getType()
                          : LLVM::LLVMVoidType::get(rewriter.getContext());
    LLVM::LLVMFunctionType funcType =
        LLVM::LLVMFunctionType::get(resultType, paramTypes);

```
- **EN**: Implements logic around `guard`, `setInsertionPoint`, `cloneInto`, `copy`, and 2 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `guard`, `setInsertionPoint`, `cloneInto`, `copy`, and 2 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

### Lines 431-447
```cpp
    LLVM::LLVMFuncOp func =
        LLVM::LLVMFuncOp::create(rewriter, loc, funcName, funcType);
    func.setAlwaysInline(true);
    rewriter.inlineRegionBefore(clonedRegion, func.getRegion(),
                                func.getRegion().end());
    for (auto &block : func.getRegion().getBlocks()) {
      if (isa<omp::YieldOp>(block.getTerminator())) {
        omp::YieldOp yieldOp = cast<omp::YieldOp>(block.getTerminator());
        rewriter.setInsertionPoint(yieldOp);
        rewriter.replaceOpWithNewOp<LLVM::ReturnOp>(yieldOp, TypeRange(),
                                                    yieldOp.getOperands());
      }
    }
    return func;
  }
};
} // namespace
```
- **EN**: Implements logic around `create`, `setAlwaysInline`, `inlineRegionBefore`, `getRegion`, and 4 more symbols; this block uses rewrite-pattern infrastructure to transform operations.
- **CN**: 围绕 `create`, `setAlwaysInline`, `inlineRegionBefore`, `getRegion`, and 4 more symbols 实现具体逻辑；该代码块使用重写模式基础设施变换操作。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Pass infrastructure / Pass 基础设施**:
  - **EN**: Wraps transformations as reusable passes that can be scheduled in MLIR pipelines.
  - **CN**: 将变换封装为可在 MLIR 流水线中调度的可复用 Pass。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Analysis/SliceAnalysis.h`, `mlir/Dialect/LLVMIR/FunctionCallUtils.h`, `mlir/Dialect/LLVMIR/LLVMDialect.h`, `mlir/Dialect/OpenMP/OpenMPDialect.h`, `mlir/IR/Builders.h`, `mlir/IR/Dominance.h`, `mlir/IR/IRMapping.h`, `mlir/Pass/Pass.h`, `mlir/Support/LLVM.h`, `llvm/Support/DebugLog.h` ... (+2 more)
- **Standard-library headers / 标准库头文件**: `<cstdint>`, `<iterator>`, `<utility>`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (4), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (3), LLVM support-library facilities / LLVM Support 库设施 (2), pass infrastructure and registration support / Pass 基础设施与注册支持 (1), support utilities and LLVM interop helpers / 支持工具与 LLVM 互操作辅助逻辑 (1)
