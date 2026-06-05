# RISCVPromoteConstant.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVPromoteConstant.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements promotion of constants into cheaper-to-materialize RISC-V forms. / 实现将常量提升为更易在 RISC-V 上物化的形式。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: File prologue and imported dependencies / 文件前言与导入依赖
```cpp
//==- RISCVPromoteConstant.cpp - Promote constant fp to global for RISC-V --==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "RISCV.h"
#include "RISCVSubtarget.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/CodeGen/TargetLowering.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/Constant.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/GlobalValue.h"
```
**EN:** This opening block combines banner comments with the first wave of includes, giving readers both context and the main compile-time dependencies.

**CN:** 该开头区段同时包含说明性注释与首批 include，让读者先看到文件定位，再看到主要编译期依赖。

### Lines 21-33: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp
#include "llvm/IR/GlobalVariable.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/InstIterator.h"
#include "llvm/IR/Instruction.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/IntrinsicInst.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/Type.h"
#include "llvm/InitializePasses.h"
#include "llvm/Pass.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/Debug.h"
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 34-43: Header guard and interface framing / 头文件保护与接口框架
```cpp
using namespace llvm;

#define DEBUG_TYPE "riscv-promote-const"
#define RISCV_PROMOTE_CONSTANT_NAME "RISC-V Promote Constants"

STATISTIC(NumPromoted, "Number of constant literals promoted to globals");
STATISTIC(NumPromotedUses, "Number of uses of promoted literal constants");

namespace {
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 44-54: Type declaration for RISCVPromoteConstant / RISCVPromoteConstant 的类型声明
```cpp
class RISCVPromoteConstant : public ModulePass {
public:
  static char ID;
  RISCVPromoteConstant() : ModulePass(ID) {}

  StringRef getPassName() const override { return RISCV_PROMOTE_CONSTANT_NAME; }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<TargetPassConfig>();
    AU.setPreservesCFG();
  }
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 55-69: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  /// Iterate over the functions and promote the double fp constants that
  /// would otherwise go into the constant pool to a constant array.
  bool runOnModule(Module &M) override {
    if (skipModule(M))
      return false;
    // TargetMachine and Subtarget are needed to query isFPImmlegal.
    const TargetPassConfig &TPC = getAnalysis<TargetPassConfig>();
    const TargetMachine &TM = TPC.getTM<TargetMachine>();
    bool Changed = false;
    for (Function &F : M) {
      const RISCVSubtarget &ST = TM.getSubtarget<RISCVSubtarget>(F);
      const RISCVTargetLowering *TLI = ST.getTargetLowering();
      Changed |= runOnFunction(F, TLI);
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 70-79: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
    return Changed;
  }

private:
  bool runOnFunction(Function &F, const RISCVTargetLowering *TLI);
};
} // end anonymous namespace

char RISCVPromoteConstant::ID = 0;
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 80-91: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
INITIALIZE_PASS(RISCVPromoteConstant, DEBUG_TYPE, RISCV_PROMOTE_CONSTANT_NAME,
                false, false)

ModulePass *llvm::createRISCVPromoteConstantPass() {
  return new RISCVPromoteConstant();
}

bool RISCVPromoteConstant::runOnFunction(Function &F,
                                         const RISCVTargetLowering *TLI) {
  if (F.hasOptNone() || F.hasOptSize())
    return false;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 92-101: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  // Bail out and make no transformation if the target doesn't support
  // doubles, or if we're not targeting RV64 as we currently see some
  // regressions for those targets.
  if (!TLI->isTypeLegal(MVT::f64) || !TLI->isTypeLegal(MVT::i64))
    return false;

  // Collect all unique double constants and their uses in the function. Use
  // MapVector to preserve insertion order.
  MapVector<ConstantFP *, SmallVector<Use *, 8>> ConstUsesMap;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 102-121: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  for (Instruction &I : instructions(F)) {
    for (Use &U : I.operands()) {
      auto *C = dyn_cast<ConstantFP>(U.get());
      if (!C || !C->getType()->isDoubleTy())
        continue;
      // Do not promote if it wouldn't be loaded from the constant pool.
      if (TLI->isFPImmLegal(C->getValueAPF(), MVT::f64,
                            /*ForCodeSize=*/false))
        continue;
      // Do not promote a constant if it is used as an immediate argument
      // for an intrinsic.
      if (auto *II = dyn_cast<IntrinsicInst>(U.getUser())) {
        Function *IntrinsicFunc = II->getFunction();
        unsigned OperandIdx = U.getOperandNo();
        if (IntrinsicFunc && IntrinsicFunc->getAttributes().hasParamAttr(
                                 OperandIdx, Attribute::ImmArg)) {
          LLVM_DEBUG(dbgs() << "Skipping promotion of constant in: " << *II
                            << " because operand " << OperandIdx
                            << " must be an immediate.\n");
          continue;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 122-139: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
        }
      }
      // Note: FP args to inline asm would be problematic if we had a
      // constraint that required an immediate floating point operand. At the
      // time of writing LLVM doesn't recognise such a constraint.
      ConstUsesMap[C].push_back(&U);
    }
  }

  int PromotableConstants = ConstUsesMap.size();
  LLVM_DEBUG(dbgs() << "Found " << PromotableConstants
                    << " promotable constants in " << F.getName() << "\n");
  // Bail out if no promotable constants found, or if only one is found.
  if (PromotableConstants < 2) {
    LLVM_DEBUG(dbgs() << "Performing no promotions as insufficient promotable "
                         "constants found\n");
    return false;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 140-150: Definitions and supporting logic / 定义与支撑逻辑
```cpp

  NumPromoted += PromotableConstants;

  // Create a global array containing the promoted constants.
  Module *M = F.getParent();
  Type *DoubleTy = Type::getDoubleTy(M->getContext());

  SmallVector<Constant *, 16> ConstantVector;
  for (auto const &Pair : ConstUsesMap)
    ConstantVector.push_back(Pair.first);
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 151-162: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  ArrayType *ArrayTy = ArrayType::get(DoubleTy, ConstantVector.size());
  Constant *GlobalArrayInitializer =
      ConstantArray::get(ArrayTy, ConstantVector);

  auto *GlobalArray = new GlobalVariable(
      *M, ArrayTy,
      /*isConstant=*/true, GlobalValue::InternalLinkage, GlobalArrayInitializer,
      ".promoted_doubles." + F.getName());

  // A cache to hold the loaded value for a given constant within a basic block.
  DenseMap<std::pair<ConstantFP *, BasicBlock *>, Value *> LocalLoads;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 163-172: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // Replace all uses with the loaded value.
  unsigned Idx = 0;
  for (auto const &Pair : ConstUsesMap) {
    ConstantFP *Const = Pair.first;
    const SmallVector<Use *, 8> &Uses = Pair.second;

    for (Use *U : Uses) {
      Instruction *UserInst = cast<Instruction>(U->getUser());
      BasicBlock *InsertionBB;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 173-185: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
      // If the user is a PHI node, we must insert the load in the
      // corresponding predecessor basic block. Otherwise, it's inserted into
      // the same block as the use.
      if (auto *PN = dyn_cast<PHINode>(UserInst))
        InsertionBB = PN->getIncomingBlock(*U);
      else
        InsertionBB = UserInst->getParent();

      if (isa<CatchSwitchInst>(InsertionBB->getTerminator())) {
        LLVM_DEBUG(dbgs() << "Bailing out: catchswitch means thre is no valid "
                             "insertion point.\n");
        return false;
      }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 186-201: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

      auto CacheKey = std::make_pair(Const, InsertionBB);
      Value *LoadedVal = nullptr;

      // Re-use a load if it exists in the insertion block.
      if (LocalLoads.count(CacheKey)) {
        LoadedVal = LocalLoads.at(CacheKey);
      } else {
        // Otherwise, create a new GEP and Load at the correct insertion point.
        // It is always safe to insert in the first insertion point in the BB,
        // so do that and let other passes reorder.
        IRBuilder<> Builder(InsertionBB, InsertionBB->getFirstInsertionPt());
        Value *ElementPtr = Builder.CreateConstInBoundsGEP2_64(
            GlobalArray->getValueType(), GlobalArray, 0, Idx, "double.addr");
        LoadedVal = Builder.CreateLoad(DoubleTy, ElementPtr, "double.val");
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 202-211: Definitions and supporting logic / 定义与支撑逻辑
```cpp
        // Cache the newly created load for this block.
        LocalLoads[CacheKey] = LoadedVal;
      }

      U->set(LoadedVal);
      ++NumPromotedUses;
    }
    ++Idx;
  }
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 212-213: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  return true;
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

## Key Concepts / 关键概念
- **RISC-V backend structure** / **RISC-V 后端结构**

## Dependencies / 依赖关系
- `RISCV.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVSubtarget.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/ADT/DenseMap.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/ADT/SmallVector.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/ADT/Statistic.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/TargetLowering.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/TargetPassConfig.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/BasicBlock.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/Constant.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/Constants.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/Function.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/IR/GlobalValue.h` — Directly referenced by this file. / 该文件直接引用的依赖。
