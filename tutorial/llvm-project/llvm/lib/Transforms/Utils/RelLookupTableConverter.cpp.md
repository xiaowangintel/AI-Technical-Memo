# RelLookupTableConverter.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/RelLookupTableConverter.cpp` | `llvm/lib/Transforms/Utils/RelLookupTableConverter.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements ===- RelLookupTableConverterPass - Rel Table Conv === within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 RelLookupTableConverter 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-28

```cpp
//===- RelLookupTableConverterPass - Rel Table Conv -----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements relative lookup table converter that converts
// lookup tables to relative lookup tables to make them PIC-friendly.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/RelLookupTableConverter.h"
#include "llvm/Analysis/ConstantFolding.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Module.h"

using namespace llvm;

struct LookupTableInfo {
  Value *Index;
  SmallVector<Constant *> Ptrs;
};

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include LookupTableInfo, which carry the local state, declarations, or transformation entry points for this region.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 LookupTableInfo，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 29-56

```cpp
static bool shouldConvertToRelLookupTable(LookupTableInfo &Info, Module &M,
                                          GlobalVariable &GV) {
  // If lookup table has more than one user,
  // do not generate a relative lookup table.
  // This is to simplify the analysis that needs to be done for this pass.
  // TODO: Add support for lookup tables with multiple uses.
  // For ex, this can happen when a function that uses a lookup table gets
  // inlined into multiple call sites.
  //
  // If the original lookup table does not have local linkage and is
  // not dso_local, do not generate a relative lookup table.
  // This optimization creates a relative lookup table that consists of
  // offsets between the start of the lookup table and its elements.
  // To be able to generate these offsets, relative lookup table and
  // its elements should have internal linkage and be dso_local, which means
  // that they should resolve to symbols within the same linkage unit.
  if (!GV.hasInitializer() || !GV.isConstant() || !GV.hasOneUse() ||
      !GV.hasLocalLinkage() || !GV.isDSOLocal() || !GV.isImplicitDSOLocal())
    return false;

  auto *GEP = dyn_cast<GetElementPtrInst>(GV.use_begin()->getUser());
  if (!GEP || !GEP->hasOneUse())
    return false;

  auto *Load = dyn_cast<LoadInst>(GEP->use_begin()->getUser());
  if (!Load || !Load->hasOneUse())
    return false;

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 57-82

```cpp
  // If values are not 64-bit pointers, do not generate a relative lookup table.
  const DataLayout &DL = M.getDataLayout();
  Type *ElemType = Load->getType();
  if (!ElemType->isPointerTy() || DL.getPointerTypeSizeInBits(ElemType) != 64)
    return false;

  // Make sure this is a gep of the form GV + scale*var.
  unsigned IndexWidth =
      DL.getIndexTypeSizeInBits(Load->getPointerOperand()->getType());
  SmallMapVector<Value *, APInt, 4> VarOffsets;
  APInt ConstOffset(IndexWidth, 0);
  if (!GEP->collectOffset(DL, IndexWidth, VarOffsets, ConstOffset) ||
      !ConstOffset.isZero() || VarOffsets.size() != 1)
    return false;

  // This can't be a pointer lookup table if the stride is smaller than a
  // pointer.
  Info.Index = VarOffsets.front().first;
  const APInt &Stride = VarOffsets.front().second;
  if (Stride.ult(DL.getTypeStoreSize(ElemType)))
    return false;

  SmallVector<GlobalVariable *, 4> GVOps;
  Triple TT = M.getTargetTriple();
  // FIXME: This should be removed in the future.
  bool ShouldDropUnnamedAddr =
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 83-108

```cpp
      // Drop unnamed_addr to avoid matching pattern in
      // `handleIndirectSymViaGOTPCRel`, which generates GOTPCREL relocations
      // not supported by the GNU linker and LLD versions below 18 on aarch64.
      TT.isAArch64()
      // Apple's ld64 (and ld-prime on Xcode 15.2) miscompile something on
      // x86_64-apple-darwin. See
      // https://github.com/rust-lang/rust/issues/140686 and
      // https://github.com/rust-lang/rust/issues/141306.
      || (TT.isX86() && TT.isOSDarwin());

  APInt Offset(IndexWidth, 0);
  uint64_t GVSize = GV.getGlobalSize(DL);
  for (; Offset.ult(GVSize); Offset += Stride) {
    Constant *C =
        ConstantFoldLoadFromConst(GV.getInitializer(), ElemType, Offset, DL);
    if (!C)
      return false;

    GlobalValue *GVOp;
    APInt GVOffset;

    // If an operand is not a constant offset from a lookup table,
    // do not generate a relative lookup table.
    if (!IsConstantOffsetFromGlobal(C, GVOp, GVOffset, DL))
      return false;

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 109-138

```cpp
    // If operand is mutable, do not generate a relative lookup table.
    auto *GlovalVarOp = dyn_cast<GlobalVariable>(GVOp);
    if (!GlovalVarOp || !GlovalVarOp->isConstant())
      return false;

    if (!GlovalVarOp->hasLocalLinkage() ||
        !GlovalVarOp->isDSOLocal() ||
        !GlovalVarOp->isImplicitDSOLocal())
      return false;

    if (ShouldDropUnnamedAddr)
      GVOps.push_back(GlovalVarOp);

    Info.Ptrs.push_back(C);
  }

  if (ShouldDropUnnamedAddr)
    for (auto *GVOp : GVOps)
      GVOp->setUnnamedAddr(GlobalValue::UnnamedAddr::None);

  return true;
}

static GlobalVariable *createRelLookupTable(LookupTableInfo &Info,
                                            Function &Func,
                                            GlobalVariable &LookupTable) {
  Module &M = *Func.getParent();
  ArrayType *IntArrayTy =
      ArrayType::get(Type::getInt32Ty(M.getContext()), Info.Ptrs.size());

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 139-165

```cpp
  GlobalVariable *RelLookupTable = new GlobalVariable(
      M, IntArrayTy, LookupTable.isConstant(), LookupTable.getLinkage(),
      nullptr, LookupTable.getName() + ".rel", &LookupTable,
      LookupTable.getThreadLocalMode(), LookupTable.getAddressSpace(),
      LookupTable.isExternallyInitialized());

  uint64_t Idx = 0;
  SmallVector<Constant *, 64> RelLookupTableContents(Info.Ptrs.size());

  for (Constant *Element : Info.Ptrs) {
    Type *IntPtrTy = M.getDataLayout().getIntPtrType(M.getContext());
    Constant *Base = llvm::ConstantExpr::getPtrToInt(RelLookupTable, IntPtrTy);
    Constant *Target = llvm::ConstantExpr::getPtrToInt(Element, IntPtrTy);
    Constant *Sub = llvm::ConstantExpr::getSub(Target, Base);
    Constant *RelOffset =
        llvm::ConstantExpr::getTrunc(Sub, Type::getInt32Ty(M.getContext()));
    RelLookupTableContents[Idx++] = RelOffset;
  }

  Constant *Initializer =
      ConstantArray::get(IntArrayTy, RelLookupTableContents);
  RelLookupTable->setInitializer(Initializer);
  RelLookupTable->setUnnamedAddr(GlobalValue::UnnamedAddr::Global);
  RelLookupTable->setAlignment(llvm::Align(4));
  return RelLookupTable;
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。

### Lines 166-193

```cpp
static void convertToRelLookupTable(LookupTableInfo &Info,
                                    GlobalVariable &LookupTable) {
  GetElementPtrInst *GEP =
      cast<GetElementPtrInst>(LookupTable.use_begin()->getUser());
  LoadInst *Load = cast<LoadInst>(GEP->use_begin()->getUser());

  Module &M = *LookupTable.getParent();
  BasicBlock *BB = GEP->getParent();
  IRBuilder<> Builder(BB);
  Function &Func = *BB->getParent();

  // Generate an array that consists of relative offsets.
  GlobalVariable *RelLookupTable =
      createRelLookupTable(Info, Func, LookupTable);

  // Place new instruction sequence before GEP.
  Builder.SetInsertPoint(GEP);
  IntegerType *IntTy = cast<IntegerType>(Info.Index->getType());
  Value *Offset = Builder.CreateShl(Info.Index, ConstantInt::get(IntTy, 2),
                                    "reltable.shift");

  // Insert the call to load.relative intrinsic before LOAD.
  // GEP might not be immediately followed by a LOAD, like it can be hoisted
  // outside the loop or another instruction might be inserted them in between.
  Builder.SetInsertPoint(Load);
  Function *LoadRelIntrinsic = llvm::Intrinsic::getOrInsertDeclaration(
      &M, Intrinsic::load_relative, {Info.Index->getType()});

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 194-220

```cpp
  // Create a call to load.relative intrinsic that computes the target address
  // by adding base address (lookup table address) and relative offset.
  Value *Result = Builder.CreateCall(LoadRelIntrinsic, {RelLookupTable, Offset},
                                     "reltable.intrinsic");

  // Replace load instruction with the new generated instruction sequence.
  Load->replaceAllUsesWith(Result);
  // Remove Load and GEP instructions.
  Load->eraseFromParent();
  GEP->eraseFromParent();
}

// Convert lookup tables to relative lookup tables in the module.
static bool convertToRelativeLookupTables(
    Module &M, function_ref<TargetTransformInfo &(Function &)> GetTTI) {
  for (Function &F : M) {
    if (F.isDeclaration())
      continue;

    // Check if we have a target that supports relative lookup tables.
    if (!GetTTI(F).shouldBuildRelLookupTables())
      return false;

    // We assume that the result is independent of the checked function.
    break;
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 221-247

```cpp
  bool Changed = false;

  for (GlobalVariable &GV : llvm::make_early_inc_range(M.globals())) {
    LookupTableInfo Info;
    if (!shouldConvertToRelLookupTable(Info, M, GV))
      continue;

    convertToRelLookupTable(Info, GV);

    // Remove the original lookup table.
    GV.eraseFromParent();

    Changed = true;
  }

  return Changed;
}

PreservedAnalyses RelLookupTableConverterPass::run(Module &M,
                                                   ModuleAnalysisManager &AM) {
  FunctionAnalysisManager &FAM =
      AM.getResult<FunctionAnalysisManagerModuleProxy>(M).getManager();

  auto GetTTI = [&](Function &F) -> TargetTransformInfo & {
    return FAM.getResult<TargetIRAnalysis>(F);
  };

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 248-254

```cpp
  if (!convertToRelativeLookupTables(M, GetTTI))
    return PreservedAnalyses::all();

  PreservedAnalyses PA;
  PA.preserveSet<CFGAnalyses>();
  return PA;
}
```
- EN: This region continues the RelLookupTableConverter implementation with local helper logic centered on GetTTI, PreservedAnalyses, CFGAnalyses.
- CN: 这一段延续了 RelLookupTableConverter 的主体实现，围绕 GetTTI, PreservedAnalyses, CFGAnalyses 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `LookupTableInfo` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`LookupTableInfo` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout, TargetTransformInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout, TargetTransformInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/ConstantFolding.h`, `llvm/Analysis/TargetTransformInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/ConstantFolding.h`, `llvm/Analysis/TargetTransformInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instructions.h`, `llvm/IR/Module.h`, `llvm/Transforms/Utils/RelLookupTableConverter.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instructions.h`, `llvm/IR/Module.h`, `llvm/Transforms/Utils/RelLookupTableConverter.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Runtime analysis dependencies: `DataLayout`, `TargetTransformInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout`, `TargetTransformInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
