# PPCBoolRetToInt.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCBoolRetToInt.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides backend implementation logic for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCBoolRetToInt.cpp`，主要负责 PowerPC 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===- PPCBoolRetToInt.cpp ------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-31

```cpp
//===----------------------------------------------------------------------===//
//
// This file implements converting i1 values to i32/i64 if they could be more
// profitably allocated as GPRs rather than CRs. This pass will become totally
// unnecessary if Register Bank Allocation and Global Instruction Selection ever
// go upstream.
//
// Presently, the pass converts i1 Constants, and Arguments to i32/i64 if the
// transitive closure of their uses includes only PHINodes, CallInsts, and
// ReturnInsts. The rational is that arguments are generally passed and returned
// in GPRs rather than CRs, so casting them to i32/i64 at the LLVM IR level will
// actually save casts at the Machine Instruction level.
//
// It might be useful to expand this pass to add bit-wise operations to the list
// of safe transitive closure types. Also, we miss some opportunities when LLVM
// represents logical AND and OR operations with control flow rather than data
// flow. For example by lowering the expression: return (A && B && C)
//
// as: return A ? true : B && C.
//
// There's code in SimplifyCFG that code be used to turn control flow in data
// flow using SelectInsts. Selects are slow on some architectures (P7/P8), so
// this probably isn't good in general, but for the special case of i1, the
// Selects could be further lowered to bit operations that are fast everywhere.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file implements converting i1 values to i32/i64 if they could be more". Notable symbols in this range include `architectures`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file implements converting i1 values to i32/i64 if they could be more”。 该区间中较显眼的符号包括 `architectures`。

### Lines 32-38

```cpp
//===----------------------------------------------------------------------===//

#include "PPC.h"
#include "PPCTargetMachine.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/SmallPtrSet.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 39-45

```cpp
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/IR/Argument.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/IRBuilder.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 46-52

```cpp
#include "llvm/IR/Instruction.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Type.h"
#include "llvm/IR/Use.h"
#include "llvm/IR/User.h"
#include "llvm/IR/Value.h"
#include "llvm/Pass.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 53-59

```cpp
#include "llvm/Support/Casting.h"
#include <cassert>

using namespace llvm;

namespace {
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 60-68

```cpp
#define DEBUG_TYPE "ppc-bool-ret-to-int"

STATISTIC(NumBoolRetPromotion,
          "Number of times a bool feeding a RetInst was promoted to an int");
STATISTIC(NumBoolCallPromotion,
          "Number of times a bool feeding a CallInst was promoted to an int");
STATISTIC(NumBoolToIntPromotion,
          "Total number of times a bool was promoted to an int");
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `STATISTIC`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `STATISTIC`。

### Lines 69-106

```cpp
class PPCBoolRetToInt : public FunctionPass {
  static SmallPtrSet<Value *, 8> findAllDefs(Value *V) {
    SmallPtrSet<Value *, 8> Defs;
    SmallVector<Value *, 8> WorkList;
    WorkList.push_back(V);
    Defs.insert(V);
    while (!WorkList.empty()) {
      Value *Curr = WorkList.pop_back_val();
      auto *CurrUser = dyn_cast<User>(Curr);
      // Operands of CallInst/Constant are skipped because they may not be Bool
      // type. For CallInst, their positions are defined by ABI.
      if (CurrUser && !isa<CallInst>(Curr) && !isa<Constant>(Curr))
        for (auto &Op : CurrUser->operands())
          if (Defs.insert(Op).second)
            WorkList.push_back(Op);
    }
    return Defs;
  }

  // Translate a i1 value to an equivalent i32/i64 value:
  Value *translate(Value *V) {
    assert(V->getType() == Type::getInt1Ty(V->getContext()) &&
           "Expect an i1 value");

    Type *IntTy = ST->isPPC64() ? Type::getInt64Ty(V->getContext())
                                : Type::getInt32Ty(V->getContext());

    if (auto *P = dyn_cast<PHINode>(V)) {
      // Temporarily set the operands to 0. We'll fix this later in
      // runOnUse.
      Value *Zero = Constant::getNullValue(IntTy);
      PHINode *Q =
        PHINode::Create(IntTy, P->getNumIncomingValues(), P->getName(), P->getIterator());
      for (unsigned I = 0; I < P->getNumOperands(); ++I)
        Q->addIncoming(Zero, P->getIncomingBlock(I));
      return Q;
    }
```
- **EN**: Declares a backend-facing type `PPCBoolRetToInt`, `findAllDefs`, `push_back` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `PPCBoolRetToInt`, `findAllDefs`, `push_back`，并勾勒出周边代码会依赖的接口或状态。

### Lines 107-144

```cpp
    IRBuilder IRB(V->getContext());
    if (auto *I = dyn_cast<Instruction>(V))
      IRB.SetInsertPoint(I->getNextNode());
    else
      IRB.SetInsertPoint(&Func->getEntryBlock(), Func->getEntryBlock().begin());
    return IRB.CreateZExt(V, IntTy);
  }

  typedef SmallPtrSet<const PHINode *, 8> PHINodeSet;

  // A PHINode is Promotable if:
  // 1. Its type is i1 AND
  // 2. All of its uses are ReturnInt, CallInst, or PHINode
  // AND
  // 3. All of its operands are Constant or Argument or
  //    CallInst or PHINode AND
  // 4. All of its PHINode uses are Promotable AND
  // 5. All of its PHINode operands are Promotable
  static PHINodeSet getPromotablePHINodes(const Function &F) {
    PHINodeSet Promotable;
    // Condition 1
    for (auto &BB : F)
      for (auto &I : BB)
        if (const auto *P = dyn_cast<PHINode>(&I))
          if (P->getType()->isIntegerTy(1))
            Promotable.insert(P);

    SmallVector<const PHINode *, 8> ToRemove;
    for (const PHINode *P : Promotable) {
      // Condition 2 and 3
      auto IsValidUser = [] (const Value *V) -> bool {
        return isa<ReturnInst>(V) || isa<CallInst>(V) || isa<PHINode>(V);
      };
      auto IsValidOperand = [] (const Value *V) -> bool {
        return isa<Constant>(V) || isa<Argument>(V) || isa<CallInst>(V) ||
        isa<PHINode>(V);
      };
      const auto &Users = P->users();
```
- **EN**: Implements helper routine(s) `IRB`, `getContext`, `SetInsertPoint` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `IRB`, `getContext`, `SetInsertPoint`。

### Lines 145-182

```cpp
      const auto &Operands = P->operands();
      if (!llvm::all_of(Users, IsValidUser) ||
          !llvm::all_of(Operands, IsValidOperand))
        ToRemove.push_back(P);
    }

    // Iterate to convergence
    auto IsPromotable = [&Promotable] (const Value *V) -> bool {
      const auto *Phi = dyn_cast<PHINode>(V);
      return !Phi || Promotable.count(Phi);
    };
    while (!ToRemove.empty()) {
      for (auto &User : ToRemove)
        Promotable.erase(User);
      ToRemove.clear();

      for (const PHINode *P : Promotable) {
        // Condition 4 and 5
        const auto &Users = P->users();
        const auto &Operands = P->operands();
        if (!llvm::all_of(Users, IsPromotable) ||
            !llvm::all_of(Operands, IsPromotable))
          ToRemove.push_back(P);
      }
    }

    return Promotable;
  }

  typedef DenseMap<Value *, Value *> B2IMap;

 public:
  static char ID;

  PPCBoolRetToInt() : FunctionPass(ID) {}

  bool runOnFunction(Function &F) override {
    if (skipFunction(F))
```
- **EN**: Implements helper routine(s) `operands`, `all_of`, `push_back` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `operands`, `all_of`, `push_back`。

### Lines 183-220

```cpp
      return false;

    auto *TPC = getAnalysisIfAvailable<TargetPassConfig>();
    if (!TPC)
      return false;

    auto &TM = TPC->getTM<PPCTargetMachine>();
    ST = TM.getSubtargetImpl(F);
    Func = &F;

    PHINodeSet PromotablePHINodes = getPromotablePHINodes(F);
    B2IMap Bool2IntMap;
    bool Changed = false;
    for (auto &BB : F) {
      for (auto &I : BB) {
        if (auto *R = dyn_cast<ReturnInst>(&I))
          if (F.getReturnType()->isIntegerTy(1))
            Changed |=
              runOnUse(R->getOperandUse(0), PromotablePHINodes, Bool2IntMap);

        if (auto *CI = dyn_cast<CallInst>(&I))
          for (auto &U : CI->operands())
            if (U->getType()->isIntegerTy(1))
              Changed |= runOnUse(U, PromotablePHINodes, Bool2IntMap);
      }
    }

    return Changed;
  }

  bool runOnUse(Use &U, const PHINodeSet &PromotablePHINodes,
                       B2IMap &BoolToIntMap) {
    auto Defs = findAllDefs(U);

    // If the values are all Constants or Arguments, don't bother
    if (llvm::none_of(Defs, [](Value *V) { return isa<Instruction>(V); }))
      return false;
```
- **EN**: Implements helper routine(s) `getSubtargetImpl`, `getPromotablePHINodes`, `getReturnType` for this portion of the PowerPC backend backend implementation logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getSubtargetImpl`, `getPromotablePHINodes`, `getReturnType`。 子目标特性裁剪会影响这里的行为。

### Lines 221-258

```cpp
    // Presently, we only know how to handle PHINode, Constant, Arguments and
    // CallInst. Potentially, bitwise operations (AND, OR, XOR, NOT) and sign
    // extension could also be handled in the future.
    for (Value *V : Defs)
      if (!isa<PHINode>(V) && !isa<Constant>(V) &&
          !isa<Argument>(V) && !isa<CallInst>(V))
        return false;

    for (Value *V : Defs)
      if (const auto *P = dyn_cast<PHINode>(V))
        if (!PromotablePHINodes.count(P))
          return false;

    if (isa<ReturnInst>(U.getUser()))
      ++NumBoolRetPromotion;
    if (isa<CallInst>(U.getUser()))
      ++NumBoolCallPromotion;
    ++NumBoolToIntPromotion;

    for (Value *V : Defs) {
      auto [It, Inserted] = BoolToIntMap.try_emplace(V);
      if (Inserted)
        It->second = translate(V);
    }

    // Replace the operands of the translated instructions. They were set to
    // zero in the translate function.
    for (auto &Pair : BoolToIntMap) {
      auto *First = dyn_cast<User>(Pair.first);
      auto *Second = dyn_cast<User>(Pair.second);
      assert((!First || Second) && "translated from user to non-user!?");
      // Operands of CallInst/Constant are skipped because they may not be Bool
      // type. For CallInst, their positions are defined by ABI.
      if (First && !isa<CallInst>(First) && !isa<Constant>(First))
        for (unsigned I = 0; I < First->getNumOperands(); ++I)
          Second->setOperand(I, BoolToIntMap[First->getOperand(I)]);
    }
```
- **EN**: Implements helper routine(s) `operations`, `count`, `getUser` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `operations`, `count`, `getUser`。

### Lines 259-286

```cpp
    Value *IntRetVal = BoolToIntMap[U];
    Type *Int1Ty = Type::getInt1Ty(U->getContext());
    auto *I = cast<Instruction>(U.getUser());
    Value *BackToBool =
        new TruncInst(IntRetVal, Int1Ty, "backToBool", I->getIterator());
    U.set(BackToBool);

    return true;
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addPreserved<DominatorTreeWrapperPass>();
    FunctionPass::getAnalysisUsage(AU);
  }

private:
  const PPCSubtarget *ST;
  Function *Func;
};

} // end anonymous namespace

char PPCBoolRetToInt::ID = 0;
INITIALIZE_PASS(PPCBoolRetToInt, "ppc-bool-ret-to-int",
                "Convert i1 constants to i32/i64 if they are returned", false,
                false)

FunctionPass *llvm::createPPCBoolRetToIntPass() { return new PPCBoolRetToInt(); }
```
- **EN**: Implements helper routine(s) `getInt1Ty`, `getContext`, `getUser` for this portion of the PowerPC backend backend implementation logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getInt1Ty`, `getContext`, `getUser`。 子目标特性裁剪会影响这里的行为。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Backend implementation logic / 后端实现逻辑
- Subtarget features / 子目标特性
- Target machine configuration / 目标机器配置
- Declarative TableGen records / 声明式 TableGen 记录
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPC.h`
- `PPCTargetMachine.h`
- `llvm/ADT/DenseMap.h`
- `llvm/ADT/STLExtras.h`
- `llvm/ADT/SmallPtrSet.h`
- `llvm/ADT/SmallVector.h`
- `llvm/ADT/Statistic.h`
- `llvm/CodeGen/TargetPassConfig.h`
- `llvm/IR/Argument.h`
- `llvm/IR/Dominators.h`
- `llvm/IR/Function.h`
- `llvm/IR/IRBuilder.h`
- `llvm/IR/Instruction.h`
- `llvm/IR/Instructions.h`
- `llvm/IR/Type.h`
- `llvm/IR/Use.h`
- `llvm/IR/User.h`
- `llvm/IR/Value.h`
- `llvm/Pass.h`
- `llvm/Support/Casting.h`
- `cassert`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM Support utilities / LLVM Support 工具层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
