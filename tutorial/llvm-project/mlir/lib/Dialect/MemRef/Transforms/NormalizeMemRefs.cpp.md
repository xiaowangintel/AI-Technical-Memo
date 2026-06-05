# NormalizeMemRefs.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/MemRef/Transforms/NormalizeMemRefs.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements an interprocedural pass to normalize memrefs to have identity layout maps.
  - **CN**: 该文件位于 `mlir/lib/Dialect/MemRef/Transforms`，围绕 MemRef 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===- NormalizeMemRefs.cpp -----------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements an interprocedural pass to normalize memrefs to have
// identity layout maps.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 13-26
```cpp

#include "mlir/Dialect/Affine/Utils.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/MemRef/IR/MemRef.h"
#include "mlir/Dialect/MemRef/Transforms/Passes.h"
#include "llvm/Support/Debug.h"

namespace mlir {
namespace memref {
#define GEN_PASS_DEF_NORMALIZEMEMREFSPASS
#include "mlir/Dialect/MemRef/Transforms/Passes.h.inc"
} // namespace memref
} // namespace mlir

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Affine/Utils.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/MemRef/IR/MemRef.h`, `mlir/Dialect/MemRef/Transforms/Passes.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Affine/Utils.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/MemRef/IR/MemRef.h`, `mlir/Dialect/MemRef/Transforms/Passes.h`。

### Lines 27-36
```cpp
#define DEBUG_TYPE "normalize-memrefs"

using namespace mlir;
using namespace mlir::affine;
using namespace mlir::memref;

namespace {

/// All memrefs passed across functions with non-trivial layout maps are
/// converted to ones with trivial identity layout ones.
```
- **EN**: Defines preprocessor-controlled structure, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成式钩子或编译期常量。

### Lines 37-52
```cpp
/// If all the memref types/uses in a function are normalizable, we treat
/// such functions as normalizable. Also, if a normalizable function is known
/// to call a non-normalizable function, we treat that function as
/// non-normalizable as well. We assume external functions to be normalizable.
struct NormalizeMemRefs
    : public memref::impl::NormalizeMemRefsPassBase<NormalizeMemRefs> {
  void runOnOperation() override;
  void normalizeFuncOpMemRefs(func::FuncOp funcOp, ModuleOp moduleOp);
  bool areMemRefsNormalizable(func::FuncOp funcOp);
  void updateFunctionSignature(func::FuncOp funcOp, ModuleOp moduleOp);
  void setCalleesAndCallersNonNormalizable(
      func::FuncOp funcOp, ModuleOp moduleOp,
      DenseSet<func::FuncOp> &normalizableFuncs);
  Operation *createOpResultsNormalized(func::FuncOp funcOp, Operation *oldOp);
};

```
- **EN**: Introduces declarations for `NormalizeMemRefs`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `NormalizeMemRefs` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 53-66
```cpp
} // namespace

void NormalizeMemRefs::runOnOperation() {
  LLVM_DEBUG(llvm::dbgs() << "Normalizing Memrefs...\n");
  ModuleOp moduleOp = getOperation();
  // We maintain all normalizable FuncOps in a DenseSet. It is initialized
  // with all the functions within a module and then functions which are not
  // normalizable are removed from this set.
  // TODO: Change this to work on FuncLikeOp once there is an operation
  // interface for it.
  DenseSet<func::FuncOp> normalizableFuncs;
  // Initialize `normalizableFuncs` with all the functions within a module.
  moduleOp.walk([&](func::FuncOp funcOp) { normalizableFuncs.insert(funcOp); });

```
- **EN**: Implements logic around `runOnOperation`, `dbgs`, `getOperation`, `walk`; this block packages logic as an MLIR pass or pass helper; expresses reusable interface-based behavior.
- **CN**: 围绕 `runOnOperation`, `dbgs`, `getOperation`, `walk` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件，并表达基于接口的可复用行为。

### Lines 67-86
```cpp
  // Traverse through all the functions applying a filter which determines
  // whether that function is normalizable or not. All callers/callees of
  // a non-normalizable function will also become non-normalizable even if
  // they aren't passing any or specific non-normalizable memrefs. So,
  // functions which calls or get called by a non-normalizable becomes non-
  // normalizable functions themselves.
  moduleOp.walk([&](func::FuncOp funcOp) {
    if (normalizableFuncs.contains(funcOp)) {
      if (!areMemRefsNormalizable(funcOp)) {
        LLVM_DEBUG(llvm::dbgs()
                   << "@" << funcOp.getName()
                   << " contains ops that cannot normalize MemRefs\n");
        // Since this function is not normalizable, we set all the caller
        // functions and the callees of this function as not normalizable.
        // TODO: Drop this conservative assumption in the future.
        setCalleesAndCallersNonNormalizable(funcOp, moduleOp,
                                            normalizableFuncs);
      }
    }
  });
```
- **EN**: Implements logic around `walk`, `contains`, `areMemRefsNormalizable`, `dbgs`, and 2 more symbols.
- **CN**: 围绕 `walk`, `contains`, `areMemRefsNormalizable`, `dbgs`, and 2 more symbols 实现具体逻辑。

### Lines 87-96
```cpp

  LLVM_DEBUG(llvm::dbgs() << "Normalizing " << normalizableFuncs.size()
                          << " functions\n");
  // Those functions which can be normalized are subjected to normalization.
  for (func::FuncOp &funcOp : normalizableFuncs)
    normalizeFuncOpMemRefs(funcOp, moduleOp);
}

/// Check whether all the uses of oldMemRef are either dereferencing uses or the
/// op is of type : DeallocOp, CallOp or ReturnOp. Only if these constraints
```
- **EN**: Implements logic around `dbgs`, `normalizeFuncOpMemRefs`.
- **CN**: 围绕 `dbgs`, `normalizeFuncOpMemRefs` 实现具体逻辑。

### Lines 97-106
```cpp
/// are satisfied will the value become a candidate for replacement.
/// TODO: Extend this for DimOps.
static bool isMemRefNormalizable(Value::user_range opUsers) {
  return llvm::all_of(opUsers, [](Operation *op) {
    return op->hasTrait<OpTrait::MemRefsNormalizable>();
  });
}

/// Set all the calling functions and the callees of the function as not
/// normalizable.
```
- **EN**: Implements logic around `isMemRefNormalizable`, `all_of`, `MemRefsNormalizable>`.
- **CN**: 围绕 `isMemRefNormalizable`, `all_of`, `MemRefsNormalizable>` 实现具体逻辑。

### Lines 107-126
```cpp
void NormalizeMemRefs::setCalleesAndCallersNonNormalizable(
    func::FuncOp funcOp, ModuleOp moduleOp,
    DenseSet<func::FuncOp> &normalizableFuncs) {
  if (!normalizableFuncs.contains(funcOp))
    return;

  LLVM_DEBUG(
      llvm::dbgs() << "@" << funcOp.getName()
                   << " calls or is called by non-normalizable function\n");
  normalizableFuncs.erase(funcOp);
  // Caller of the function.
  std::optional<SymbolTable::UseRange> symbolUses =
      funcOp.getSymbolUses(moduleOp);
  for (SymbolTable::SymbolUse symbolUse : *symbolUses) {
    // TODO: Extend this for ops that are FunctionOpInterface. This would
    // require creating an OpInterface for FunctionOpInterface ops.
    func::FuncOp parentFuncOp =
        symbolUse.getUser()->getParentOfType<func::FuncOp>();
    for (func::FuncOp &funcOp : normalizableFuncs) {
      if (parentFuncOp == funcOp) {
```
- **EN**: Implements logic around `setCalleesAndCallersNonNormalizable`, `contains`, `dbgs`, `erase`, and 2 more symbols.
- **CN**: 围绕 `setCalleesAndCallersNonNormalizable`, `contains`, `dbgs`, `erase`, and 2 more symbols 实现具体逻辑。

### Lines 127-146
```cpp
        setCalleesAndCallersNonNormalizable(funcOp, moduleOp,
                                            normalizableFuncs);
        break;
      }
    }
  }

  // Functions called by this function.
  funcOp.walk([&](func::CallOp callOp) {
    StringAttr callee = callOp.getCalleeAttr().getAttr();
    for (func::FuncOp &funcOp : normalizableFuncs) {
      // We compare func::FuncOp and callee's name.
      if (callee == funcOp.getNameAttr()) {
        setCalleesAndCallersNonNormalizable(funcOp, moduleOp,
                                            normalizableFuncs);
        break;
      }
    }
  });
}
```
- **EN**: Implements logic around `setCalleesAndCallersNonNormalizable`, `walk`, `getCalleeAttr`, `getNameAttr`.
- **CN**: 围绕 `setCalleesAndCallersNonNormalizable`, `walk`, `getCalleeAttr`, `getNameAttr` 实现具体逻辑。

### Lines 147-159
```cpp

/// Check whether all the uses of AllocOps, AllocaOps, CallOps and function
/// arguments of a function are either of dereferencing type or are uses in:
/// DeallocOp, CallOp or ReturnOp. Only if these constraints are satisfied will
/// the function become a candidate for normalization. When the uses of a memref
/// are non-normalizable and the memref map layout is trivial (identity), we can
/// still label the entire function as normalizable. We assume external
/// functions to be normalizable.
bool NormalizeMemRefs::areMemRefsNormalizable(func::FuncOp funcOp) {
  // We assume external functions to be normalizable.
  if (funcOp.isExternal())
    return true;

```
- **EN**: Implements logic around `areMemRefsNormalizable`, `isExternal`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `areMemRefsNormalizable`, `isExternal` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 160-170
```cpp
  if (funcOp
          .walk([&](AllocOp allocOp) -> WalkResult {
            Value oldMemRef = allocOp.getResult();
            if (!allocOp.getType().getLayout().isIdentity() &&
                !isMemRefNormalizable(oldMemRef.getUsers()))
              return WalkResult::interrupt();
            return WalkResult::advance();
          })
          .wasInterrupted())
    return false;

```
- **EN**: Implements logic around `walk`, `getResult`, `getType`, `isMemRefNormalizable`, and 3 more symbols.
- **CN**: 围绕 `walk`, `getResult`, `getType`, `isMemRefNormalizable`, and 3 more symbols 实现具体逻辑。

### Lines 171-181
```cpp
  if (funcOp
          .walk([&](AllocaOp allocaOp) -> WalkResult {
            Value oldMemRef = allocaOp.getResult();
            if (!allocaOp.getType().getLayout().isIdentity() &&
                !isMemRefNormalizable(oldMemRef.getUsers()))
              return WalkResult::interrupt();
            return WalkResult::advance();
          })
          .wasInterrupted())
    return false;

```
- **EN**: Implements logic around `walk`, `getResult`, `getType`, `isMemRefNormalizable`, and 3 more symbols.
- **CN**: 围绕 `walk`, `getResult`, `getType`, `isMemRefNormalizable`, and 3 more symbols 实现具体逻辑。

### Lines 182-197
```cpp
  if (funcOp
          .walk([&](func::CallOp callOp) -> WalkResult {
            for (unsigned resIndex :
                 llvm::seq<unsigned>(0, callOp.getNumResults())) {
              Value oldMemRef = callOp.getResult(resIndex);
              if (auto oldMemRefType =
                      dyn_cast<MemRefType>(oldMemRef.getType()))
                if (!oldMemRefType.getLayout().isIdentity() &&
                    !isMemRefNormalizable(oldMemRef.getUsers()))
                  return WalkResult::interrupt();
            }
            return WalkResult::advance();
          })
          .wasInterrupted())
    return false;

```
- **EN**: Implements logic around `walk`, `seq`, `getResult`, `getType`, and 5 more symbols.
- **CN**: 围绕 `walk`, `seq`, `getResult`, `getType`, and 5 more symbols 实现具体逻辑。

### Lines 198-208
```cpp
  for (unsigned argIndex : llvm::seq<unsigned>(0, funcOp.getNumArguments())) {
    BlockArgument oldMemRef = funcOp.getArgument(argIndex);
    if (auto oldMemRefType = dyn_cast<MemRefType>(oldMemRef.getType()))
      if (!oldMemRefType.getLayout().isIdentity() &&
          !isMemRefNormalizable(oldMemRef.getUsers()))
        return false;
  }

  return true;
}

```
- **EN**: Implements logic around `seq`, `getArgument`, `getType`, `getLayout`, and 1 more symbols.
- **CN**: 围绕 `seq`, `getArgument`, `getType`, `getLayout`, and 1 more symbols 实现具体逻辑。

### Lines 209-221
```cpp
/// Fetch the updated argument list and result of the function and update the
/// function signature. This updates the function's return type at the caller
/// site and in case the return type is a normalized memref then it updates
/// the calling function's signature.
/// TODO: An update to the calling function signature is required only if the
/// returned value is in turn used in ReturnOp of the calling function.
void NormalizeMemRefs::updateFunctionSignature(func::FuncOp funcOp,
                                               ModuleOp moduleOp) {
  FunctionType functionType = funcOp.getFunctionType();
  SmallVector<Type, 4> resultTypes;
  FunctionType newFuncType;
  resultTypes = llvm::to_vector<4>(functionType.getResults());

```
- **EN**: Implements logic around `updateFunctionSignature`, `getFunctionType`, `to_vector`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `updateFunctionSignature`, `getFunctionType`, `to_vector` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 222-241
```cpp
  // External function's signature was already updated in
  // 'normalizeFuncOpMemRefs()'.
  if (!funcOp.isExternal()) {
    SmallVector<Type, 8> argTypes;
    for (const auto &argEn : llvm::enumerate(funcOp.getArguments()))
      argTypes.push_back(argEn.value().getType());

    // Traverse ReturnOps to check if an update to the return type in the
    // function signature is required.
    funcOp.walk([&](func::ReturnOp returnOp) {
      for (const auto &operandEn : llvm::enumerate(returnOp.getOperands())) {
        Type opType = operandEn.value().getType();
        MemRefType memrefType = dyn_cast<MemRefType>(opType);
        // If type is not memref or if the memref type is same as that in
        // function's return signature then no update is required.
        if (!memrefType || memrefType == resultTypes[operandEn.index()])
          continue;
        // Update function's return type signature.
        // Return type gets normalized either as a result of function argument
        // normalization, AllocOp normalization or an update made at CallOp.
```
- **EN**: Implements logic around `isExternal`, `enumerate`, `push_back`, `walk`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `isExternal`, `enumerate`, `push_back`, `walk`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 242-251
```cpp
        // There can be many call flows inside a function and an update to a
        // specific ReturnOp has not yet been made. So we check that the result
        // memref type is normalized.
        // TODO: When selective normalization is implemented, handle multiple
        // results case where some are normalized, some aren't.
        if (memrefType.getLayout().isIdentity())
          resultTypes[operandEn.index()] = memrefType;
      }
    });

```
- **EN**: Implements logic around `getLayout`, `index`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getLayout`, `index` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 252-271
```cpp
    // We create a new function type and modify the function signature with this
    // new type.
    newFuncType = FunctionType::get(&getContext(), /*inputs=*/argTypes,
                                    /*results=*/resultTypes);
  }

  // Since we update the function signature, it might affect the result types at
  // the caller site. Since this result might even be used by the caller
  // function in ReturnOps, the caller function's signature will also change.
  // Hence we record the caller function in 'funcOpsToUpdate' to update their
  // signature as well.
  llvm::SmallDenseSet<func::FuncOp, 8> funcOpsToUpdate;
  // We iterate over all symbolic uses of the function and update the return
  // type at the caller site.
  std::optional<SymbolTable::UseRange> symbolUses =
      funcOp.getSymbolUses(moduleOp);
  for (SymbolTable::SymbolUse symbolUse : *symbolUses) {
    Operation *userOp = symbolUse.getUser();
    OpBuilder builder(userOp);
    // When `userOp` can not be casted to `CallOp`, it is skipped. This assumes
```
- **EN**: Implements logic around `get`, `getSymbolUses`, `getUser`, `builder`.
- **CN**: 围绕 `get`, `getSymbolUses`, `getUser`, `builder` 实现具体逻辑。

### Lines 272-291
```cpp
    // that the non-CallOp has no memrefs to be replaced.
    // TODO: Handle cases where a non-CallOp symbol use of a function deals with
    // memrefs.
    auto callOp = dyn_cast<func::CallOp>(userOp);
    if (!callOp)
      continue;
    Operation *newCallOp =
        func::CallOp::create(builder, userOp->getLoc(), callOp.getCalleeAttr(),
                             resultTypes, userOp->getOperands());
    bool replacingMemRefUsesFailed = false;
    bool returnTypeChanged = false;
    for (unsigned resIndex : llvm::seq<unsigned>(0, userOp->getNumResults())) {
      OpResult oldResult = userOp->getResult(resIndex);
      OpResult newResult = newCallOp->getResult(resIndex);
      // This condition ensures that if the result is not of type memref or if
      // the resulting memref was already having a trivial map layout then we
      // need not perform any use replacement here.
      if (oldResult.getType() == newResult.getType())
        continue;
      AffineMap layoutMap =
```
- **EN**: Implements logic around `CallOp>`, `create`, `getOperands`, `seq`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `CallOp>`, `create`, `getOperands`, `seq`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 292-311
```cpp
          cast<MemRefType>(oldResult.getType()).getLayout().getAffineMap();
      if (failed(replaceAllMemRefUsesWith(oldResult, /*newMemRef=*/newResult,
                                          /*extraIndices=*/{},
                                          /*indexRemap=*/layoutMap,
                                          /*extraOperands=*/{},
                                          /*symbolOperands=*/{},
                                          /*userFilterFn=*/nullptr,
                                          /*allowNonDereferencingOps=*/true,
                                          /*replaceInDeallocOp=*/true))) {
        // If it failed (due to escapes for example), bail out.
        // It should never hit this part of the code because it is called by
        // only those functions which are normalizable.
        newCallOp->erase();
        replacingMemRefUsesFailed = true;
        break;
      }
      returnTypeChanged = true;
    }
    if (replacingMemRefUsesFailed)
      continue;
```
- **EN**: Implements logic around `getType`, `failed`, `erase`.
- **CN**: 围绕 `getType`, `failed`, `erase` 实现具体逻辑。

### Lines 312-331
```cpp
    // Replace all uses for other non-memref result types.
    userOp->replaceAllUsesWith(newCallOp);
    userOp->erase();
    if (returnTypeChanged) {
      // Since the return type changed it might lead to a change in function's
      // signature.
      // TODO: If funcOp doesn't return any memref type then no need to update
      // signature.
      // TODO: Further optimization - Check if the memref is indeed part of
      // ReturnOp at the parentFuncOp and only then updation of signature is
      // required.
      // TODO: Extend this for ops that are FunctionOpInterface. This would
      // require creating an OpInterface for FunctionOpInterface ops.
      func::FuncOp parentFuncOp = newCallOp->getParentOfType<func::FuncOp>();
      funcOpsToUpdate.insert(parentFuncOp);
    }
  }
  // Because external function's signature is already updated in
  // 'normalizeFuncOpMemRefs()', we don't need to update it here again.
  if (!funcOp.isExternal())
```
- **EN**: Implements logic around `replaceAllUsesWith`, `erase`, `FuncOp>`, `insert`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `replaceAllUsesWith`, `erase`, `FuncOp>`, `insert`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 332-341
```cpp
    funcOp.setType(newFuncType);

  // Updating the signature type of those functions which call the current
  // function. Only if the return type of the current function has a normalized
  // memref will the caller function become a candidate for signature update.
  for (func::FuncOp parentFuncOp : funcOpsToUpdate)
    updateFunctionSignature(parentFuncOp, moduleOp);
}

/// Normalizes the memrefs within a function which includes those arising as a
```
- **EN**: Implements logic around `setType`, `updateFunctionSignature`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `setType`, `updateFunctionSignature` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 342-361
```cpp
/// result of AllocOps, AllocaOps, CallOps, ReinterpretCastOps and function's
/// argument. The ModuleOp argument is used to help update function's signature
/// after normalization.
void NormalizeMemRefs::normalizeFuncOpMemRefs(func::FuncOp funcOp,
                                              ModuleOp moduleOp) {
  // Turn memrefs' non-identity layouts maps into ones with identity. Collect
  // alloc, alloca ops and reinterpret_cast ops first and then process since
  // normalizeMemRef replaces/erases ops during memref rewriting.
  SmallVector<AllocOp, 4> allocOps;
  SmallVector<AllocaOp> allocaOps;
  SmallVector<ReinterpretCastOp> reinterpretCastOps;
  funcOp.walk([&](Operation *op) {
    if (auto allocOp = dyn_cast<AllocOp>(op))
      allocOps.push_back(allocOp);
    else if (auto allocaOp = dyn_cast<AllocaOp>(op))
      allocaOps.push_back(allocaOp);
    else if (auto reinterpretCastOp = dyn_cast<ReinterpretCastOp>(op))
      reinterpretCastOps.push_back(reinterpretCastOp);
  });
  for (AllocOp allocOp : allocOps)
```
- **EN**: Implements logic around `normalizeFuncOpMemRefs`, `walk`, `push_back`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `normalizeFuncOpMemRefs`, `walk`, `push_back` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 362-381
```cpp
    (void)normalizeMemRef(allocOp);
  for (AllocaOp allocaOp : allocaOps)
    (void)normalizeMemRef(allocaOp);
  for (ReinterpretCastOp reinterpretCastOp : reinterpretCastOps)
    (void)normalizeMemRef(reinterpretCastOp);

  // We use this OpBuilder to create new memref layout later.
  OpBuilder b(funcOp);

  FunctionType functionType = funcOp.getFunctionType();
  SmallVector<Location> functionArgLocs(llvm::map_range(
      funcOp.getArguments(), [](BlockArgument arg) { return arg.getLoc(); }));
  SmallVector<Type, 8> inputTypes;
  // Walk over each argument of a function to perform memref normalization (if
  for (unsigned argIndex :
       llvm::seq<unsigned>(0, functionType.getNumInputs())) {
    Type argType = functionType.getInput(argIndex);
    MemRefType memrefType = dyn_cast<MemRefType>(argType);
    // Check whether argument is of MemRef type. Any other argument type can
    // simply be part of the final function signature.
```
- **EN**: Implements logic around `normalizeMemRef`, `b`, `getFunctionType`, `functionArgLocs`, and 3 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `normalizeMemRef`, `b`, `getFunctionType`, `functionArgLocs`, and 3 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 382-395
```cpp
    if (!memrefType) {
      inputTypes.push_back(argType);
      continue;
    }
    // Fetch a new memref type after normalizing the old memref to have an
    // identity map layout.
    MemRefType newMemRefType = normalizeMemRefType(memrefType);
    if (newMemRefType == memrefType || funcOp.isExternal()) {
      // Either memrefType already had an identity map or the map couldn't be
      // transformed to an identity map.
      inputTypes.push_back(newMemRefType);
      continue;
    }

```
- **EN**: Implements logic around `push_back`, `normalizeMemRefType`, `isExternal`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `push_back`, `normalizeMemRefType`, `isExternal` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 396-415
```cpp
    // Insert a new temporary argument with the new memref type.
    BlockArgument newMemRef = funcOp.front().insertArgument(
        argIndex, newMemRefType, functionArgLocs[argIndex]);
    BlockArgument oldMemRef = funcOp.getArgument(argIndex + 1);
    AffineMap layoutMap = memrefType.getLayout().getAffineMap();
    // Replace all uses of the old memref.
    if (failed(replaceAllMemRefUsesWith(oldMemRef, /*newMemRef=*/newMemRef,
                                        /*extraIndices=*/{},
                                        /*indexRemap=*/layoutMap,
                                        /*extraOperands=*/{},
                                        /*symbolOperands=*/{},
                                        /*userFilterFn=*/nullptr,
                                        /*allowNonDereferencingOps=*/true,
                                        /*replaceInDeallocOp=*/true))) {
      // If it failed (due to escapes for example), bail out. Removing the
      // temporary argument inserted previously.
      funcOp.front().eraseArgument(argIndex);
      continue;
    }

```
- **EN**: Implements logic around `front`, `getArgument`, `getLayout`, `failed`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `front`, `getArgument`, `getLayout`, `failed` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 416-435
```cpp
    // All uses for the argument with old memref type were replaced
    // successfully. So we remove the old argument now.
    funcOp.front().eraseArgument(argIndex + 1);
  }

  // Walk over normalizable operations to normalize memrefs of the operation
  // results. When `op` has memrefs with affine map in the operation results,
  // new operation containin normalized memrefs is created. Then, the memrefs
  // are replaced. `CallOp` is skipped here because it is handled in
  // `updateFunctionSignature()`.
  funcOp.walk([&](Operation *op) {
    if (op->hasTrait<OpTrait::MemRefsNormalizable>() &&
        op->getNumResults() > 0 && !isa<func::CallOp>(op) &&
        !funcOp.isExternal()) {
      // Create newOp containing normalized memref in the operation result.
      Operation *newOp = createOpResultsNormalized(funcOp, op);
      // When all of the operation results have no memrefs or memrefs without
      // affine map, `newOp` is the same with `op` and following process is
      // skipped.
      if (op != newOp) {
```
- **EN**: Implements logic around `front`, `walk`, `MemRefsNormalizable>`, `getNumResults`, and 2 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `front`, `walk`, `MemRefsNormalizable>`, `getNumResults`, and 2 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 436-455
```cpp
        bool replacingMemRefUsesFailed = false;
        for (unsigned resIndex : llvm::seq<unsigned>(0, op->getNumResults())) {
          // Replace all uses of the old memrefs.
          Value oldMemRef = op->getResult(resIndex);
          Value newMemRef = newOp->getResult(resIndex);
          MemRefType oldMemRefType = dyn_cast<MemRefType>(oldMemRef.getType());
          // Check whether the operation result is MemRef type.
          if (!oldMemRefType)
            continue;
          MemRefType newMemRefType = cast<MemRefType>(newMemRef.getType());
          if (oldMemRefType == newMemRefType)
            continue;
          // TODO: Assume single layout map. Multiple maps not supported.
          AffineMap layoutMap = oldMemRefType.getLayout().getAffineMap();
          if (failed(replaceAllMemRefUsesWith(oldMemRef,
                                              /*newMemRef=*/newMemRef,
                                              /*extraIndices=*/{},
                                              /*indexRemap=*/layoutMap,
                                              /*extraOperands=*/{},
                                              /*symbolOperands=*/{},
```
- **EN**: Implements logic around `seq`, `getResult`, `getType`, `getLayout`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `seq`, `getResult`, `getType`, `getLayout`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 456-473
```cpp
                                              /*userFilterFn=*/nullptr,
                                              /*allowNonDereferencingOps=*/true,
                                              /*replaceInDeallocOp=*/true))) {
            newOp->erase();
            replacingMemRefUsesFailed = true;
            continue;
          }
        }
        if (!replacingMemRefUsesFailed) {
          // Replace other ops with new op and delete the old op when the
          // replacement succeeded.
          op->replaceAllUsesWith(newOp);
          op->erase();
        }
      }
    }
  });

```
- **EN**: Implements logic around `erase`, `replaceAllUsesWith`.
- **CN**: 围绕 `erase`, `replaceAllUsesWith` 实现具体逻辑。

### Lines 474-493
```cpp
  // In a normal function, memrefs in the return type signature gets normalized
  // as a result of normalization of functions arguments, AllocOps or CallOps'
  // result types. Since an external function doesn't have a body, memrefs in
  // the return type signature can only get normalized by iterating over the
  // individual return types.
  if (funcOp.isExternal()) {
    SmallVector<Type, 4> resultTypes;
    for (unsigned resIndex :
         llvm::seq<unsigned>(0, functionType.getNumResults())) {
      Type resType = functionType.getResult(resIndex);
      MemRefType memrefType = dyn_cast<MemRefType>(resType);
      // Check whether result is of MemRef type. Any other argument type can
      // simply be part of the final function signature.
      if (!memrefType) {
        resultTypes.push_back(resType);
        continue;
      }
      // Computing a new memref type after normalizing the old memref to have an
      // identity map layout.
      MemRefType newMemRefType = normalizeMemRefType(memrefType);
```
- **EN**: Implements logic around `isExternal`, `seq`, `getResult`, `push_back`, and 1 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `isExternal`, `seq`, `getResult`, `push_back`, and 1 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 494-505
```cpp
      resultTypes.push_back(newMemRefType);
    }

    FunctionType newFuncType =
        FunctionType::get(&getContext(), /*inputs=*/inputTypes,
                          /*results=*/resultTypes);
    // Setting the new function signature for this external function.
    funcOp.setType(newFuncType);
  }
  updateFunctionSignature(funcOp, moduleOp);
}

```
- **EN**: Implements logic around `push_back`, `get`, `setType`, `updateFunctionSignature`.
- **CN**: 围绕 `push_back`, `get`, `setType`, `updateFunctionSignature` 实现具体逻辑。

### Lines 506-525
```cpp
/// Create an operation containing normalized memrefs in the operation results.
/// When the results of `oldOp` have memrefs with affine map, the memrefs are
/// normalized, and new operation containing them in the operation results is
/// returned. If all of the results of `oldOp` have no memrefs or memrefs
/// without affine map, `oldOp` is returned without modification.
Operation *NormalizeMemRefs::createOpResultsNormalized(func::FuncOp funcOp,
                                                       Operation *oldOp) {
  // Prepare OperationState to create newOp containing normalized memref in
  // the operation results.
  OperationState result(oldOp->getLoc(), oldOp->getName());
  result.addOperands(oldOp->getOperands());
  result.addAttributes(oldOp->getAttrs());
  // Add normalized MemRefType to the OperationState.
  SmallVector<Type, 4> resultTypes;
  OpBuilder b(funcOp);
  bool resultTypeNormalized = false;
  for (unsigned resIndex : llvm::seq<unsigned>(0, oldOp->getNumResults())) {
    auto resultType = oldOp->getResult(resIndex).getType();
    MemRefType memrefType = dyn_cast<MemRefType>(resultType);
    // Check whether the operation result is MemRef type.
```
- **EN**: Implements logic around `createOpResultsNormalized`, `result`, `addOperands`, `addAttributes`, and 3 more symbols; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `createOpResultsNormalized`, `result`, `addOperands`, `addAttributes`, and 3 more symbols 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 526-545
```cpp
    if (!memrefType) {
      resultTypes.push_back(resultType);
      continue;
    }

    // Fetch a new memref type after normalizing the old memref.
    MemRefType newMemRefType = normalizeMemRefType(memrefType);
    if (newMemRefType == memrefType) {
      // Either memrefType already had an identity map or the map couldn't
      // be transformed to an identity map.
      resultTypes.push_back(memrefType);
      continue;
    }
    resultTypes.push_back(newMemRefType);
    resultTypeNormalized = true;
  }
  result.addTypes(resultTypes);
  // When all of the results of `oldOp` have no memrefs or memrefs without
  // affine map, `oldOp` is returned without modification.
  if (resultTypeNormalized) {
```
- **EN**: Implements logic around `push_back`, `normalizeMemRefType`, `addTypes`; this block coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `push_back`, `normalizeMemRefType`, `addTypes` 实现具体逻辑；该代码块协调核心结构化 MLIR 方言之间的行为。

### Lines 546-554
```cpp
    OpBuilder bb(oldOp);
    for (auto &oldRegion : oldOp->getRegions()) {
      Region *newRegion = result.addRegion();
      newRegion->takeBody(oldRegion);
    }
    return bb.create(result);
  }
  return oldOp;
}
```
- **EN**: Implements logic around `bb`, `getRegions`, `addRegion`, `takeBody`, and 1 more symbols.
- **CN**: 围绕 `bb`, `getRegions`, `addRegion`, `takeBody`, and 1 more symbols 实现具体逻辑。

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

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Affine/Utils.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/MemRef/IR/MemRef.h`, `mlir/Dialect/MemRef/Transforms/Passes.h`, `llvm/Support/Debug.h`, `mlir/Dialect/MemRef/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (5), LLVM support-library facilities / LLVM Support 库设施 (1)
