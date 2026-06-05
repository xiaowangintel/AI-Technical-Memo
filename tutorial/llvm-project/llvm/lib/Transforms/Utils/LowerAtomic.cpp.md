# LowerAtomic.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/LowerAtomic.cpp` | `llvm/lib/Transforms/Utils/LowerAtomic.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements lower atomic intrinsics within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 LowerAtomic 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-17

```cpp
//===- LowerAtomic.cpp - Lower atomic intrinsics --------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass lowers atomic intrinsics to non-atomic form for use in a known
// non-preemptible environment.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/LowerAtomic.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/IRBuilder.h"

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 18-34

```cpp
using namespace llvm;

#define DEBUG_TYPE "loweratomic"

bool llvm::lowerAtomicCmpXchgInst(AtomicCmpXchgInst *CXI) {
  IRBuilder<> Builder(CXI);
  Value *Ptr = CXI->getPointerOperand();
  Value *Cmp = CXI->getCompareOperand();
  Value *Val = CXI->getNewValOperand();

  auto [Orig, Equal] =
      buildCmpXchgValue(Builder, Ptr, Cmp, Val, CXI->getAlign());

  Value *Res =
      Builder.CreateInsertValue(PoisonValue::get(CXI->getType()), Orig, 0);
  Res = Builder.CreateInsertValue(Res, Equal, 1);

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics. Core entities appearing here include lowerAtomicCmpXchgInst, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。 此处出现的核心实体包括 lowerAtomicCmpXchgInst，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 35-51

```cpp
  CXI->replaceAllUsesWith(Res);
  CXI->eraseFromParent();
  return true;
}

std::pair<Value *, Value *> llvm::buildCmpXchgValue(IRBuilderBase &Builder,
                                                    Value *Ptr, Value *Cmp,
                                                    Value *Val,
                                                    Align Alignment) {
  LoadInst *Orig = Builder.CreateAlignedLoad(Val->getType(), Ptr, Alignment);
  Value *Equal = Builder.CreateICmpEQ(Orig, Cmp);
  Value *Res = Builder.CreateSelect(Equal, Val, Orig);
  Builder.CreateAlignedStore(Res, Ptr, Alignment);

  return {Orig, Equal};
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 52-66

```cpp
Value *llvm::buildAtomicRMWValue(AtomicRMWInst::BinOp Op,
                                 IRBuilderBase &Builder, Value *Loaded,
                                 Value *Val) {
  Value *NewVal;
  switch (Op) {
  case AtomicRMWInst::Xchg:
    return Val;
  case AtomicRMWInst::Add:
    return Builder.CreateAdd(Loaded, Val, "new");
  case AtomicRMWInst::Sub:
    return Builder.CreateSub(Loaded, Val, "new");
  case AtomicRMWInst::And:
    return Builder.CreateAnd(Loaded, Val, "new");
  case AtomicRMWInst::Nand:
    return Builder.CreateNot(Builder.CreateAnd(Loaded, Val), "new");
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 67-81

```cpp
  case AtomicRMWInst::Or:
    return Builder.CreateOr(Loaded, Val, "new");
  case AtomicRMWInst::Xor:
    return Builder.CreateXor(Loaded, Val, "new");
  case AtomicRMWInst::Max:
    NewVal = Builder.CreateICmpSGT(Loaded, Val);
    return Builder.CreateSelect(NewVal, Loaded, Val, "new");
  case AtomicRMWInst::Min:
    NewVal = Builder.CreateICmpSLE(Loaded, Val);
    return Builder.CreateSelect(NewVal, Loaded, Val, "new");
  case AtomicRMWInst::UMax:
    NewVal = Builder.CreateICmpUGT(Loaded, Val);
    return Builder.CreateSelect(NewVal, Loaded, Val, "new");
  case AtomicRMWInst::UMin:
    NewVal = Builder.CreateICmpULE(Loaded, Val);
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 82-96

```cpp
    return Builder.CreateSelect(NewVal, Loaded, Val, "new");
  case AtomicRMWInst::FAdd:
    return Builder.CreateFAdd(Loaded, Val, "new");
  case AtomicRMWInst::FSub:
    return Builder.CreateFSub(Loaded, Val, "new");
  case AtomicRMWInst::FMax:
    return Builder.CreateMaxNum(Loaded, Val);
  case AtomicRMWInst::FMin:
    return Builder.CreateMinNum(Loaded, Val);
  case AtomicRMWInst::FMaximum:
    return Builder.CreateMaximum(Loaded, Val);
  case AtomicRMWInst::FMinimum:
    return Builder.CreateMinimum(Loaded, Val);
  case AtomicRMWInst::FMaximumNum:
    return Builder.CreateMaximumNum(Loaded, Val);
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 97-109

```cpp
  case AtomicRMWInst::FMinimumNum:
    return Builder.CreateMinimumNum(Loaded, Val);
  case AtomicRMWInst::UIncWrap: {
    Constant *One = ConstantInt::get(Loaded->getType(), 1);
    Value *Inc = Builder.CreateAdd(Loaded, One);
    Value *Cmp = Builder.CreateICmpUGE(Loaded, Val);
    Constant *Zero = ConstantInt::get(Loaded->getType(), 0);
    return Builder.CreateSelect(Cmp, Zero, Inc, "new");
  }
  case AtomicRMWInst::UDecWrap: {
    Constant *Zero = ConstantInt::get(Loaded->getType(), 0);
    Constant *One = ConstantInt::get(Loaded->getType(), 1);

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 110-128

```cpp
    Value *Dec = Builder.CreateSub(Loaded, One);
    Value *CmpEq0 = Builder.CreateICmpEQ(Loaded, Zero);
    Value *CmpOldGtVal = Builder.CreateICmpUGT(Loaded, Val);
    Value *Or = Builder.CreateOr(CmpEq0, CmpOldGtVal);
    return Builder.CreateSelect(Or, Val, Dec, "new");
  }
  case AtomicRMWInst::USubCond: {
    Value *Cmp = Builder.CreateICmpUGE(Loaded, Val);
    Value *Sub = Builder.CreateSub(Loaded, Val);
    return Builder.CreateSelect(Cmp, Sub, Loaded, "new");
  }
  case AtomicRMWInst::USubSat:
    return Builder.CreateIntrinsic(Intrinsic::usub_sat, Loaded->getType(),
                                   {Loaded, Val}, nullptr, "new");
  default:
    llvm_unreachable("Unknown atomic op");
  }
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 129-143

```cpp
bool llvm::lowerAtomicRMWInst(AtomicRMWInst *RMWI) {
  IRBuilder<> Builder(RMWI);
  Builder.setIsFPConstrained(
      RMWI->getFunction()->hasFnAttribute(Attribute::StrictFP));

  Value *Ptr = RMWI->getPointerOperand();
  Value *Val = RMWI->getValOperand();

  LoadInst *Orig = Builder.CreateLoad(Val->getType(), Ptr);
  Value *Res = buildAtomicRMWValue(RMWI->getOperation(), Builder, Orig, Val);
  Builder.CreateStore(Res, Ptr);
  RMWI->replaceAllUsesWith(Orig);
  RMWI->eraseFromParent();
  return true;
}
```
- EN: Core entities appearing here include lowerAtomicRMWInst, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 lowerAtomicRMWInst，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `lowerAtomicCmpXchgInst, lowerAtomicRMWInst` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`lowerAtomicCmpXchgInst, lowerAtomicRMWInst` 构成该文件对外 API 与主要实现挂钩。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/Transforms/Utils/LowerAtomic.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/Transforms/Utils/LowerAtomic.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
