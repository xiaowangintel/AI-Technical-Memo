# ACCRecipeMaterialization.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/OpenACC/Transforms/ACCRecipeMaterialization.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements OpenACC dialect support for rewrite patterns, passes, and IR-to-IR transformation logic, centered on `ACCRecipeMaterialization`.
  - **CN**: 实现 OpenACC 方言中围绕 `ACCRecipeMaterialization` 的重写模式、Pass 以及 IR 到 IR 的变换逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
//===- ACCRecipeMaterialization.cpp - Materialize ACC recipes -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Overview:
// ---------
// OpenACC compute constructs (acc.parallel, acc.serial, acc.kernels) and
// acc.loop can carry data clauses (acc.private, acc.firstprivate,
// acc.reduction) that refer to recipes (acc.private.recipe,
// acc.firstprivate.recipe, acc.reduction.recipe). Recipes define how to
// initialize, copy, combine, or destroy a particular variable. This pass clones
// those regions into the construct and ensures the materialized SSA values are
// used instead.
//
// Transforms:
// -----------
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 21-40
```cpp
// 1. Firstprivate: Inserts acc.firstprivate_map so the initial value is
//    available on the device, then clones the recipe init and copy regions
//    into the construct and replaces uses with the materialized alloca.
//    Optional destroy region is cloned before the region terminator.
//
// 2. Private: Clones the recipe init region into the construct (at the
//    region entry or at the loop op for acc.loop private). Replaces uses
//    of the recipe result with the materialized alloca. Optional destroy
//    region is cloned before the region terminator.
//
// 3. Reduction: Creates acc.reduction_init (init region inlined) and
//    acc.reduction_combine_region (combiner region inlined). Uses within
//    the region are updated to the reduction init result.
//
// Requirements:
// -------------
// 1. OpenACCSupport: The pass uses the `acc::OpenACCSupport` analysis
//    including emitNYI for unsupported cases.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 41-60
```cpp

#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Arith/Utils/Utils.h"
#include "mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h"
#include "mlir/Dialect/OpenACC/OpenACC.h"
#include "mlir/Dialect/OpenACC/OpenACCUtils.h"
#include "mlir/Dialect/OpenACC/OpenACCUtilsLoop.h"
#include "mlir/Dialect/OpenACC/Transforms/Passes.h"
#include "mlir/IR/Block.h"
#include "mlir/IR/Builders.h"
#include "mlir/IR/IRMapping.h"
#include "mlir/IR/SymbolTable.h"
#include "mlir/IR/Value.h"
#include "mlir/IR/ValueRange.h"
#include "mlir/Interfaces/LoopLikeInterface.h"
#include "mlir/Support/LLVM.h"
#include "mlir/Transforms/RegionUtils.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/TypeSwitch.h"
#include "llvm/Support/Debug.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h`, `mlir/Dialect/OpenACC/OpenACC.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h`, `mlir/Dialect/OpenACC/OpenACC.h`。

### Lines 61-71
```cpp
#include "llvm/Support/ErrorHandling.h"

namespace mlir {
namespace acc {
#define GEN_PASS_DEF_ACCRECIPEMATERIALIZATION
#include "mlir/Dialect/OpenACC/Transforms/Passes.h.inc"
} // namespace acc
} // namespace mlir

#define DEBUG_TYPE "acc-recipe-materialization"

```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/Support/ErrorHandling.h`, `mlir/Dialect/OpenACC/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/Support/ErrorHandling.h`, `mlir/Dialect/OpenACC/Transforms/Passes.h.inc`。

### Lines 72-91
```cpp
namespace {

using namespace mlir;

static void saveVarName(StringRef name, Value dst) {
  if (name.empty())
    return;
  if (Operation *dstOp = dst.getDefiningOp()) {
    if (dstOp->getAttrOfType<acc::VarNameAttr>(acc::getVarNameAttrName()))
      return;
    if (isa<ACC_DATA_ENTRY_OPS>(dstOp))
      return;
    dstOp->setAttr(acc::getVarNameAttrName(),
                   acc::VarNameAttr::get(dstOp->getContext(), name));
    return;
  }
  auto blockArg = dyn_cast<BlockArgument>(dst);
  if (!blockArg)
    return;
  Block *block = blockArg.getOwner();
```
- **EN**: Introduces declarations for `mlir`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 92-109
```cpp
  Region *region = block ? block->getParent() : nullptr;
  if (!region || !block->isEntryBlock())
    return;
  Operation *parent = region->getParentOp();
  if (!parent)
    return;
  auto funcOp = dyn_cast<FunctionOpInterface>(parent);
  if (!funcOp)
    return;
  unsigned argIdx = blockArg.getArgNumber();
  if (argIdx >= funcOp.getNumArguments())
    return;
  if (funcOp.getArgAttr(argIdx, acc::getVarNameAttrName()))
    return;
  funcOp.setArgAttr(argIdx, acc::getVarNameAttrName(),
                    acc::VarNameAttr::get(parent->getContext(), name));
}

```
- **EN**: Implements logic around `getParent`, `isEntryBlock`, `getParentOp`, `getArgNumber`, and 4 more symbols.
- **CN**: 围绕 `getParent`, `isEntryBlock`, `getParentOp`, `getArgNumber`, and 4 more symbols 实现具体逻辑。

### Lines 110-129
```cpp
static void saveVarName(Value src, Value dst) {
  saveVarName(acc::getVariableName(src), dst);
}

// Clone the destroy region of the recipe before the terminator of the provided
// block. Values must be provided for the destroy region block arguments
// according to the recipe specifications.
template <typename RecipeOpTy>
static void cloneDestroy(RecipeOpTy recipe, mlir::Block *block,
                         const llvm::SmallVector<mlir::Value> &arguments) {
  IRMapping mapping{};
  Region &destroyRegion = recipe.getDestroyRegion();
  assert(destroyRegion.getBlocks().front().getNumArguments() ==
             arguments.size() &&
         "unexpected acc recipe destroy block arguments");
  mapping.map(destroyRegion.getBlocks().front().getArguments(), arguments);
  acc::cloneACCRegionInto(&destroyRegion, block, std::prev(block->end()),
                          mapping,
                          /*resultsToReplace=*/{});
}
```
- **EN**: Implements logic around `saveVarName`, `cloneDestroy`, `getDestroyRegion`, `assert`, and 3 more symbols.
- **CN**: 围绕 `saveVarName`, `cloneDestroy`, `getDestroyRegion`, `assert`, and 3 more symbols 实现具体逻辑。

### Lines 130-149
```cpp

class ACCRecipeMaterialization
    : public acc::impl::ACCRecipeMaterializationBase<ACCRecipeMaterialization> {
public:
  using acc::impl::ACCRecipeMaterializationBase<
      ACCRecipeMaterialization>::ACCRecipeMaterializationBase;
  void runOnOperation() override;

private:
  // When handling firstprivate, the initial value needs to be available on
  // the GPU. One way to get that value there is to map the variable through
  // global memory.
  // Thus, when we materialize a firstprivate, we materialize it into
  // a mapping action first. This function ends up with doing the following:
  // %dev = acc.firstprivate var(%var)
  // =>
  // %copy = acc.firstprivate_map var(%var)
  // %dev = acc.firstprivate var(%copy)
  // When the recipe materialization happens, the `acc.firstprivate` ends up
  // being removed. But because of the way we chain it to the
```
- **EN**: Introduces declarations for `ACCRecipeMaterialization`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ACCRecipeMaterialization` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 150-162
```cpp
  // `acc.firstprivate_map`, then its result becomes live-in to the
  // compute region and used as the variable the initial value is loaded from.
  void handleFirstprivateMapping(acc::FirstprivateOp firstprivateOp) const;
  template <typename OpTy>
  void removeRecipe(OpTy op, ModuleOp moduleOp) const;
  template <typename OpTy, typename RecipeOpTy, typename AccOpTy>
  LogicalResult materialize(OpTy op, RecipeOpTy recipe, AccOpTy accOp,
                            acc::OpenACCSupport &accSupport) const;
  template <typename OpTy>
  LogicalResult materializeForACCOp(OpTy accOp,
                                    acc::OpenACCSupport &accSupport) const;
};

```
- **EN**: Implements logic around `handleFirstprivateMapping`, `removeRecipe`, `materialize`, `materializeForACCOp`.
- **CN**: 围绕 `handleFirstprivateMapping`, `removeRecipe`, `materialize`, `materializeForACCOp` 实现具体逻辑。

### Lines 163-173
```cpp
void ACCRecipeMaterialization::handleFirstprivateMapping(
    acc::FirstprivateOp firstprivateOp) const {
  OpBuilder builder(firstprivateOp);
  auto mapFirstprivateOp = acc::FirstprivateMapInitialOp::create(
      builder, firstprivateOp.getLoc(), firstprivateOp.getVar(),
      firstprivateOp.getStructured(), firstprivateOp.getImplicit(),
      firstprivateOp.getBounds());
  mapFirstprivateOp.setName(firstprivateOp.getName());
  firstprivateOp.getVarMutable().assign(mapFirstprivateOp.getAccVar());
}

```
- **EN**: Implements logic around `handleFirstprivateMapping`, `builder`, `create`, `getLoc`, and 4 more symbols.
- **CN**: 围绕 `handleFirstprivateMapping`, `builder`, `create`, `getLoc`, and 4 more symbols 实现具体逻辑。

### Lines 174-193
```cpp
template <typename OpTy>
void ACCRecipeMaterialization::removeRecipe(OpTy op, ModuleOp moduleOp) const {
  auto recipeName = op.getNameAttr();
  if (SymbolTable::symbolKnownUseEmpty(recipeName, moduleOp)) {
    LLVM_DEBUG(llvm::dbgs() << "erasing recipe: " << recipeName << "\n");
    op.erase();
  } else {
    LLVM_DEBUG({
      std::optional<SymbolTable::UseRange> symbolUses =
          op.getSymbolUses(moduleOp);
      if (symbolUses.has_value()) {
        for (SymbolTable::SymbolUse symbolUse : *symbolUses) {
          llvm::dbgs() << "symbol use: ";
          symbolUse.getUser()->dump();
        }
      }
    });
    llvm_unreachable("expected no use of recipe symbol");
  }
}
```
- **EN**: Implements logic around `removeRecipe`, `getNameAttr`, `symbolKnownUseEmpty`, `dbgs`, and 4 more symbols.
- **CN**: 围绕 `removeRecipe`, `getNameAttr`, `symbolKnownUseEmpty`, `dbgs`, and 4 more symbols 实现具体逻辑。

### Lines 194-203
```cpp

template <typename OpTy, typename RecipeOpTy, typename AccOpTy>
LogicalResult
ACCRecipeMaterialization::materialize(OpTy op, RecipeOpTy recipe, AccOpTy accOp,
                                      acc::OpenACCSupport &accSupport) const {
  Region &region = accOp.getRegion();
  Value origPtr = op.getVar();
  Value accPtr = op.getAccVar();
  assert(accPtr && "invalid op: null acc var");

```
- **EN**: Implements logic around `materialize`, `getRegion`, `getVar`, `getAccVar`, and 1 more symbols.
- **CN**: 围绕 `materialize`, `getRegion`, `getVar`, `getAccVar`, and 1 more symbols 实现具体逻辑。

### Lines 204-223
```cpp
  OpBuilder b(op);
  SmallVector<Value> triples;

  // Clone init block into the region at the insertion point specified.
  Region &initRegion = recipe.getInitRegion();
  unsigned initNumArguments =
      initRegion.getBlocks().front().getArguments().size();
  if (initNumArguments > 1) {
    // Code from C/C++ will most likely only provide extent arguments to the
    // recipe arguments.
    if ((initNumArguments - 1) % 3 != 0) {
      (void)accSupport.emitNYI(recipe.getLoc(),
                               "privatization of array section with extents");
      return failure();
    }
    // The remaining arguments must be the bounds triples
    // (lower-bound, upper-bound, step), ...
    unsigned argIdx = 1;
    // Cast the given value to the type of the combiner region's argument
    // at position argIdx, and increment argIdx.
```
- **EN**: Implements logic around `b`, `getInitRegion`, `getBlocks`, `emitNYI`, and 1 more symbols.
- **CN**: 围绕 `b`, `getInitRegion`, `getBlocks`, `emitNYI`, and 1 more symbols 实现具体逻辑。

### Lines 224-243
```cpp
    auto castValueToArgType = [&](Location loc, Value v) {
      return convertScalarToDtype(
          b, loc, v,
          initRegion.getBlocks().front().getArgument(argIdx++).getType(),
          /*isUnsignedCast=*/false);
    };
    for (Value bound : acc::getBounds(op)) {
      auto dataBound = bound.getDefiningOp<acc::DataBoundsOp>();
      assert(dataBound &&
             "acc.reduction's bound must be defined by acc.bounds");
      // NOTE: we should probably generate get_lowerbound, get_upperbound
      // and get_stride here, so that we can stop looking for the acc.bounds
      // operation above, and just use the `bound` value.
      Value lb =
          castValueToArgType(dataBound.getLoc(), dataBound.getLowerbound());
      Value ub =
          castValueToArgType(dataBound.getLoc(), dataBound.getUpperbound());
      Value step =
          castValueToArgType(dataBound.getLoc(), dataBound.getStride());
      triples.append({lb, ub, step});
```
- **EN**: Implements logic around `convertScalarToDtype`, `getBlocks`, `getBounds`, `DataBoundsOp>`, and 3 more symbols.
- **CN**: 围绕 `convertScalarToDtype`, `getBlocks`, `getBounds`, `DataBoundsOp>`, and 3 more symbols 实现具体逻辑。

### Lines 244-254
```cpp
    }
    assert(triples.size() + 1 == initNumArguments &&
           "mismatch between number bounds and number of recipe init block "
           "arguments");
  }

  IRMapping mapping;
  SmallVector<Value> initArgs{origPtr};
  initArgs.append(triples);
  mapping.map(initRegion.getBlocks().front().getArguments(), initArgs);

```
- **EN**: Implements logic around `assert`, `append`, `map`.
- **CN**: 围绕 `assert`, `append`, `map` 实现具体逻辑。

### Lines 255-274
```cpp
  if constexpr (std::is_same_v<OpTy, acc::PrivateOp>) {
    // Clone the init region for a private.
    Block *block = &region.front();
    auto [results, ip] = acc::cloneACCRegionInto(
        &initRegion, block, block->begin(), mapping, {accPtr});
    assert(results.size() == 1 && "expected single result from init region");
    saveVarName(op.getAccVar(), results[0]);
    // Clone the destroy region for a private, if it exists.
    if (!recipe.getDestroyRegion().empty()) {
      results.insert(results.begin(), origPtr);
      results.append(triples);
      cloneDestroy(recipe, block, results);
    }
  } else if constexpr (std::is_same_v<OpTy, acc::FirstprivateOp>) {
    // Clone the init region for a firstprivate.
    Block *block = &region.front();
    auto [results, ip] = acc::cloneACCRegionInto(
        &initRegion, block, block->begin(), mapping, {accPtr});
    assert(results.size() == 1 && "expected single result from init region");
    saveVarName(op.getAccVar(), results[0]);
```
- **EN**: Implements logic around `constexpr`, `front`, `cloneACCRegionInto`, `begin`, and 6 more symbols.
- **CN**: 围绕 `constexpr`, `front`, `cloneACCRegionInto`, `begin`, and 6 more symbols 实现具体逻辑。

### Lines 275-294
```cpp
    // We want the copy to store the origPtr to private
    results.insert(results.begin(), origPtr);
    results.append(triples);

    // Clone the copy region for a firstprivate
    mapping.clear();
    mapping.map(recipe.getCopyRegion().front().getArguments(), results);
    // Clone the copy region for a firstprivate.
    acc::cloneACCRegionInto(&recipe.getCopyRegion(), block, std::next(ip),
                            mapping, {});
    if (!recipe.getDestroyRegion().empty()) {
      // origPtr was already pushed.
      cloneDestroy(recipe, block, results);
    }
  } else if constexpr (std::is_same_v<OpTy, acc::ReductionOp>) {
    auto cloneRegionIntoAccRegion = [&](Region *src, Region *dest,
                                        bool hasResult) {
      src->cloneInto(dest, mapping);
      Block *block = &dest->front();
      Operation *terminator = block->getTerminator();
```
- **EN**: Implements logic around `insert`, `append`, `clear`, `map`, and 7 more symbols.
- **CN**: 围绕 `insert`, `append`, `clear`, `map`, and 7 more symbols 实现具体逻辑。

### Lines 295-310
```cpp
      b.setInsertionPoint(terminator);
      if (hasResult)
        acc::YieldOp::create(b, op.getLoc(), terminator->getOperands());
      else
        acc::YieldOp::create(b, op.getLoc(), ValueRange{});
      terminator->erase();
    };

    // Clone the init region into acc.reduction_init.
    if constexpr (std::is_same_v<AccOpTy, acc::ParallelOp>)
      b.setInsertionPointToStart(&region.front());
    else if constexpr (std::is_same_v<AccOpTy, acc::LoopOp>)
      b.setInsertionPoint(op);
    else
      llvm_unreachable("unexpected acc op with reduction recipe");

```
- **EN**: Implements logic around `setInsertionPoint`, `create`, `erase`, `constexpr`, and 1 more symbols.
- **CN**: 围绕 `setInsertionPoint`, `create`, `erase`, `constexpr`, and 1 more symbols 实现具体逻辑。

### Lines 311-323
```cpp
    auto reductionOp = acc::ReductionInitOp::create(
        b, op.getLoc(), origPtr, recipe.getReductionOperatorAttr());
    saveVarName(op.getAccVar(), reductionOp.getResult());
    cloneRegionIntoAccRegion(&initRegion, &reductionOp.getRegion(),
                             /*hasResult=*/true);

    // Update the uses within the loop to use the reduction op result.
    replaceAllUsesInRegionWith(accPtr, reductionOp.getResult(), region);

    // Clone the combiner region into acc.reduction_combine_region.
    Region &combinerRegion = recipe.getCombinerRegion();
    Block *entryBlock = &combinerRegion.front();

```
- **EN**: Implements logic around `create`, `getLoc`, `saveVarName`, `cloneRegionIntoAccRegion`, and 3 more symbols.
- **CN**: 围绕 `create`, `getLoc`, `saveVarName`, `cloneRegionIntoAccRegion`, and 3 more symbols 实现具体逻辑。

### Lines 324-339
```cpp
    if constexpr (std::is_same_v<AccOpTy, acc::ParallelOp>)
      b.setInsertionPoint(region.back().getTerminator());
    else if constexpr (std::is_same_v<AccOpTy, acc::LoopOp>)
      b.setInsertionPointAfter(accOp);
    else
      llvm_unreachable("unexpected acc op with reduction recipe");

    // Map the first two block arguments to the original and private
    // reduction variables. If the recipe's combiner region has the bounds
    // arguments, we have to map them to the corresponding operands of
    // acc.reduction operation.
    mapping.clear();
    SmallVector<Value, 2> argsRemapping{origPtr, reductionOp.getResult()};
    argsRemapping.append(triples);
    mapping.map(entryBlock->getArguments(), argsRemapping);

```
- **EN**: Implements logic around `constexpr`, `setInsertionPoint`, `setInsertionPointAfter`, `clear`, and 3 more symbols.
- **CN**: 围绕 `constexpr`, `setInsertionPoint`, `setInsertionPointAfter`, `clear`, and 3 more symbols 实现具体逻辑。

### Lines 340-354
```cpp
    auto combineRegionOp = acc::ReductionCombineRegionOp::create(
        b, op.getLoc(), origPtr, reductionOp.getResult());
    cloneRegionIntoAccRegion(&combinerRegion, &combineRegionOp.getRegion(),
                             /*hasResult=*/false);

    auto setSeqParDimsForRecipeLoops = [](Region *r) {
      r->walk([](LoopLikeOpInterface loopLike) {
        loopLike->setAttr(
            acc::GPUParallelDimsAttr::name,
            acc::GPUParallelDimsAttr::seq(loopLike->getContext()));
      });
    };
    setSeqParDimsForRecipeLoops(&reductionOp.getRegion());
    setSeqParDimsForRecipeLoops(&combineRegionOp.getRegion());

```
- **EN**: Implements logic around `create`, `getLoc`, `cloneRegionIntoAccRegion`, `walk`, and 3 more symbols.
- **CN**: 围绕 `create`, `getLoc`, `cloneRegionIntoAccRegion`, `walk`, and 3 more symbols 实现具体逻辑。

### Lines 355-364
```cpp
    if (!recipe.getDestroyRegion().empty()) {
      (void)accSupport.emitNYI(
          recipe.getLoc(),
          "OpenACC reduction variable that requires destruction code");
      return failure();
    }
  } else {
    llvm_unreachable("unexpected op type");
  }

```
- **EN**: Implements logic around `getDestroyRegion`, `emitNYI`, `getLoc`, `failure`.
- **CN**: 围绕 `getDestroyRegion`, `emitNYI`, `getLoc`, `failure` 实现具体逻辑。

### Lines 365-384
```cpp
  op.erase();
  return success();
}

template <typename OpTy>
LogicalResult ACCRecipeMaterialization::materializeForACCOp(
    OpTy accOp, acc::OpenACCSupport &accSupport) const {
  assert(isa<ACC_COMPUTE_CONSTRUCT_AND_LOOP_OPS>(accOp));

  if (!accOp.getFirstprivateOperands().empty()) {
    // Clear the firstprivate operands list so there will be no uses after
    // the recipe is materialized.
    SmallVector<Value> operands(accOp.getFirstprivateOperands());
    accOp.getFirstprivateOperandsMutable().clear();
    for (Value operand : operands) {
      auto firstprivateOp = cast<acc::FirstprivateOp>(operand.getDefiningOp());
      auto symbolRef = cast<SymbolRefAttr>(firstprivateOp.getRecipeAttr());
      auto decl = SymbolTable::lookupNearestSymbolFrom(accOp, symbolRef);
      auto recipeOp = cast<acc::FirstprivateRecipeOp>(decl);
      LLVM_DEBUG(llvm::dbgs() << "materializing: " << firstprivateOp << "\n"
```
- **EN**: Implements logic around `erase`, `success`, `materializeForACCOp`, `assert`, and 8 more symbols.
- **CN**: 围绕 `erase`, `success`, `materializeForACCOp`, `assert`, and 8 more symbols 实现具体逻辑。

### Lines 385-404
```cpp
                              << symbolRef << "\n");
      handleFirstprivateMapping(firstprivateOp);
      if (failed(materialize(firstprivateOp, recipeOp, accOp, accSupport)))
        return failure();
    }
  }

  if (!accOp.getPrivateOperands().empty()) {
    // Clear the private operands list so there will be no uses after
    // the recipe is materialized.
    SmallVector<Value> operands(accOp.getPrivateOperands());
    accOp.getPrivateOperandsMutable().clear();
    for (Value operand : operands) {
      auto privateOp = cast<acc::PrivateOp>(operand.getDefiningOp());
      auto symbolRef = cast<SymbolRefAttr>(privateOp.getRecipeAttr());
      auto decl = SymbolTable::lookupNearestSymbolFrom(accOp, symbolRef);
      auto recipeOp = cast<acc::PrivateRecipeOp>(decl);
      LLVM_DEBUG(llvm::dbgs() << "materializing: " << privateOp << "\n"
                              << symbolRef << "\n");
      if (failed(materialize(privateOp, recipeOp, accOp, accSupport)))
```
- **EN**: Implements logic around `handleFirstprivateMapping`, `failed`, `failure`, `getPrivateOperands`, and 7 more symbols.
- **CN**: 围绕 `handleFirstprivateMapping`, `failed`, `failure`, `getPrivateOperands`, and 7 more symbols 实现具体逻辑。

### Lines 405-424
```cpp
        return failure();
    }
  }

  if (!accOp.getReductionOperands().empty()) {
    // Clear the reduction operands list so there will be no uses after
    // the recipe is materialized.
    SmallVector<Value> operands(accOp.getReductionOperands());
    accOp.getReductionOperandsMutable().clear();
    for (Value operand : operands) {
      auto reductionOp = cast<acc::ReductionOp>(operand.getDefiningOp());
      auto symbolRef = cast<SymbolRefAttr>(reductionOp.getRecipeAttr());
      auto decl = SymbolTable::lookupNearestSymbolFrom(accOp, symbolRef);
      auto recipeOp = cast<acc::ReductionRecipeOp>(decl);
      LLVM_DEBUG(llvm::dbgs() << "materializing: " << reductionOp << "\n"
                              << symbolRef << "\n");
      if (failed(materialize(reductionOp, recipeOp, accOp, accSupport)))
        return failure();
    }
  }
```
- **EN**: Implements logic around `failure`, `getReductionOperands`, `operands`, `getReductionOperandsMutable`, and 6 more symbols.
- **CN**: 围绕 `failure`, `getReductionOperands`, `operands`, `getReductionOperandsMutable`, and 6 more symbols 实现具体逻辑。

### Lines 425-444
```cpp
  return success();
}

void ACCRecipeMaterialization::runOnOperation() {
  ModuleOp moduleOp = getOperation();
  acc::OpenACCSupport &accSupport = getAnalysis<acc::OpenACCSupport>();

  // Materialize all recipes for all compute constructs and loop constructs.
  bool anyFailed = false;
  moduleOp.walk([&](Operation *op) {
    if (anyFailed)
      return;
    TypeSwitch<Operation *>(op).Case<ACC_COMPUTE_CONSTRUCT_AND_LOOP_OPS>(
        [&](auto constructOp) {
          if (failed(materializeForACCOp(constructOp, accSupport)))
            anyFailed = true;
        });
  });
  if (anyFailed) {
    signalPassFailure();
```
- **EN**: Implements logic around `success`, `runOnOperation`, `getOperation`, `OpenACCSupport>`, and 4 more symbols; this block packages logic as an MLIR pass or pass helper.
- **CN**: 围绕 `success`, `runOnOperation`, `getOperation`, `OpenACCSupport>`, and 4 more symbols 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件。

### Lines 445-458
```cpp
    return;
  }

  // Remove all recipes.
  moduleOp.walk([&](Operation *op) {
    if (auto recipe = dyn_cast<acc::ReductionRecipeOp>(op))
      removeRecipe(recipe, moduleOp);
    else if (auto recipe = dyn_cast<acc::PrivateRecipeOp>(op))
      removeRecipe(recipe, moduleOp);
    else if (auto recipe = dyn_cast<acc::FirstprivateRecipeOp>(op))
      removeRecipe(recipe, moduleOp);
  });
}

```
- **EN**: Implements logic around `walk`, `ReductionRecipeOp>`, `removeRecipe`, `PrivateRecipeOp>`, and 1 more symbols.
- **CN**: 围绕 `walk`, `ReductionRecipeOp>`, `removeRecipe`, `PrivateRecipeOp>`, and 1 more symbols 实现具体逻辑。

### Lines 459-459
```cpp
} // namespace
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

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
- **Target-specific semantics / 目标相关语义**:
  - **EN**: Encodes rules tied to accelerator, GPU, or binary target environments.
  - **CN**: 编码与加速器、GPU 或二进制目标环境绑定的规则。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/Arith/IR/Arith.h`, `mlir/Dialect/Arith/Utils/Utils.h`, `mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h`, `mlir/Dialect/OpenACC/OpenACC.h`, `mlir/Dialect/OpenACC/OpenACCUtils.h`, `mlir/Dialect/OpenACC/OpenACCUtilsLoop.h`, `mlir/Dialect/OpenACC/Transforms/Passes.h`, `mlir/IR/Block.h`, `mlir/IR/Builders.h`, `mlir/IR/IRMapping.h` ... (+11 more)
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (8), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (6), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (2), LLVM support-library facilities / LLVM Support 库设施 (2), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (1), support utilities and LLVM interop helpers / 支持工具与 LLVM 互操作辅助逻辑 (1), generic transformation and conversion helpers / 通用变换与转换辅助逻辑 (1)
