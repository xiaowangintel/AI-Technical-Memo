# XCoreLowerThreadLocal.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/XCore/XCoreLowerThreadLocal.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements target-specific lowering for thread-local storage accesses.
  - **CN**: 实现线程本地存储访问的目标专用 lowering。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- XCoreLowerThreadLocal - Lower thread local variables --------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-14
```cpp
///
/// \file
/// This file contains a pass that lowers thread local variables on the
///        XCore.
///
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 15-27
```cpp
#include "XCore.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/DerivedTypes.h"
#include "llvm/IR/GlobalVariable.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/Intrinsics.h"
#include "llvm/IR/IntrinsicsXCore.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/ValueHandle.h"
#include "llvm/Pass.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `XCore.h`, `llvm/IR/Constants.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/GlobalVariable.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `XCore.h`, `llvm/IR/Constants.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/GlobalVariable.h`。

### Lines 28-36
```cpp
#define DEBUG_TYPE "xcore-lower-thread-local"

using namespace llvm;

static cl::opt<unsigned> MaxThreads(
  "xcore-max-threads", cl::Optional,
  cl::desc("Maximum number of threads (for emulation thread-local storage)"),
  cl::Hidden, cl::value_desc("number"), cl::init(8));

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 37-43
```cpp
namespace {
  /// Lowers thread local variables on the XCore. Each thread local variable is
  /// expanded to an array of n elements indexed by the thread ID where n is the
  /// fixed number hardware threads supported by the device.
  struct XCoreLowerThreadLocal : public ModulePass {
    static char ID;

```
- **EN**: Introduces declarations for `XCoreLowerThreadLocal`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `XCoreLowerThreadLocal` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 44-51
```cpp
    XCoreLowerThreadLocal() : ModulePass(ID) {}

    bool lowerGlobal(GlobalVariable *GV);

    bool runOnModule(Module &M) override;
  };
}

```
- **EN**: Implements logic around `XCoreLowerThreadLocal`, `lowerGlobal`, `runOnModule`.
- **CN**: 围绕 `XCoreLowerThreadLocal`, `lowerGlobal`, `runOnModule` 实现具体逻辑。

### Lines 52-60
```cpp
char XCoreLowerThreadLocal::ID = 0;

INITIALIZE_PASS(XCoreLowerThreadLocal, "xcore-lower-thread-local",
                "Lower thread local variables", false, false)

ModulePass *llvm::createXCoreLowerThreadLocalPass() {
  return new XCoreLowerThreadLocal();
}

```
- **EN**: Implements logic around `INITIALIZE_PASS`, `createXCoreLowerThreadLocalPass`, `XCoreLowerThreadLocal`; this block returns target-specific results.
- **CN**: 围绕 `INITIALIZE_PASS`, `createXCoreLowerThreadLocalPass`, `XCoreLowerThreadLocal` 实现具体逻辑；这一段返回目标相关结果。

### Lines 61-73
```cpp
static ArrayType *createLoweredType(Type *OriginalType) {
  return ArrayType::get(OriginalType, MaxThreads);
}

static Constant *
createLoweredInitializer(ArrayType *NewType, Constant *OriginalInitializer) {
  SmallVector<Constant *, 8> Elements(MaxThreads);
  for (unsigned i = 0; i != MaxThreads; ++i) {
    Elements[i] = OriginalInitializer;
  }
  return ConstantArray::get(NewType, Elements);
}

```
- **EN**: Implements logic around `createLoweredType`, `get`, `createLoweredInitializer`, `Elements`; this block returns target-specific results.
- **CN**: 围绕 `createLoweredType`, `get`, `createLoweredInitializer`, `Elements` 实现具体逻辑；这一段返回目标相关结果。

### Lines 74-87
```cpp

static bool replaceConstantExprOp(ConstantExpr *CE, Pass *P) {
  do {
    SmallVector<WeakTrackingVH, 8> WUsers(CE->users());
    llvm::sort(WUsers);
    WUsers.erase(llvm::unique(WUsers), WUsers.end());
    while (!WUsers.empty())
      if (WeakTrackingVH WU = WUsers.pop_back_val()) {
        if (PHINode *PN = dyn_cast<PHINode>(WU)) {
          for (int I = 0, E = PN->getNumIncomingValues(); I < E; ++I)
            if (PN->getIncomingValue(I) == CE) {
              BasicBlock *PredBB = PN->getIncomingBlock(I);
              if (PredBB->getTerminator()->getNumSuccessors() > 1)
                PredBB = SplitEdge(PredBB, PN->getParent());
```
- **EN**: Implements logic around `replaceConstantExprOp`, `WUsers`, `sort`, `erase`, ...; this block applies conditional target rules.
- **CN**: 围绕 `replaceConstantExprOp`, `WUsers`, `sort`, `erase`, ... 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 88-101
```cpp
              BasicBlock::iterator InsertPos =
                  PredBB->getTerminator()->getIterator();
              Instruction *NewInst = CE->getAsInstruction();
              NewInst->insertBefore(*PredBB, InsertPos);
              PN->setOperand(I, NewInst);
            }
        } else if (Instruction *Instr = dyn_cast<Instruction>(WU)) {
          Instruction *NewInst = CE->getAsInstruction();
          NewInst->insertBefore(*Instr->getParent(), Instr->getIterator());
          Instr->replaceUsesOfWith(CE, NewInst);
        } else {
          ConstantExpr *CExpr = dyn_cast<ConstantExpr>(WU);
          if (!CExpr || !replaceConstantExprOp(CExpr, P))
            return false;
```
- **EN**: Implements logic around `getTerminator`, `getAsInstruction`, `insertBefore`, `setOperand`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getTerminator`, `getAsInstruction`, `insertBefore`, `setOperand`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 102-109
```cpp
        }
      }
  } while (CE->hasNUsesOrMore(1)); // We need to check because a recursive
  // sibling may have used 'CE' when getAsInstruction was called.
  CE->destroyConstant();
  return true;
}

```
- **EN**: Implements logic around `destroyConstant`; this block returns target-specific results.
- **CN**: 围绕 `destroyConstant` 实现具体逻辑；这一段返回目标相关结果。

### Lines 110-123
```cpp
static bool rewriteNonInstructionUses(GlobalVariable *GV, Pass *P) {
  SmallVector<WeakTrackingVH, 8> WUsers;
  for (User *U : GV->users())
    if (!isa<Instruction>(U))
      WUsers.push_back(WeakTrackingVH(U));
  while (!WUsers.empty())
    if (WeakTrackingVH WU = WUsers.pop_back_val()) {
      ConstantExpr *CE = dyn_cast<ConstantExpr>(WU);
      if (!CE || !replaceConstantExprOp(CE, P))
        return false;
    }
  return true;
}

```
- **EN**: Implements logic around `rewriteNonInstructionUses`, `push_back`, `dyn_cast<ConstantExpr>`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `rewriteNonInstructionUses`, `push_back`, `dyn_cast<ConstantExpr>` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 124-133
```cpp
static bool isZeroLengthArray(Type *Ty) {
  ArrayType *AT = dyn_cast<ArrayType>(Ty);
  return AT && (AT->getNumElements() == 0);
}

bool XCoreLowerThreadLocal::lowerGlobal(GlobalVariable *GV) {
  Module *M = GV->getParent();
  if (!GV->isThreadLocal())
    return false;

```
- **EN**: Implements logic around `isZeroLengthArray`, `dyn_cast<ArrayType>`, `getNumElements`, `lowerGlobal`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `isZeroLengthArray`, `dyn_cast<ArrayType>`, `getNumElements`, `lowerGlobal`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 134-147
```cpp
  // Skip globals that we can't lower and leave it for the backend to error.
  if (!rewriteNonInstructionUses(GV, this) ||
      !GV->getType()->isSized() || isZeroLengthArray(GV->getType()))
    return false;

  // Create replacement global.
  ArrayType *NewType = createLoweredType(GV->getValueType());
  Constant *NewInitializer = nullptr;
  if (GV->hasInitializer())
    NewInitializer = createLoweredInitializer(NewType,
                                              GV->getInitializer());
  GlobalVariable *NewGV =
    new GlobalVariable(*M, NewType, GV->isConstant(), GV->getLinkage(),
                       NewInitializer, "", nullptr,
```
- **EN**: Implements logic around `getType`, `createLoweredType`, `createLoweredInitializer`, `getInitializer`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getType`, `createLoweredType`, `createLoweredInitializer`, `getInitializer`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 148-161
```cpp
                       GlobalVariable::NotThreadLocal,
                       GV->getType()->getAddressSpace(),
                       GV->isExternallyInitialized());

  // Update uses.
  SmallVector<User *, 16> Users(GV->users());
  for (User *U : Users) {
    Instruction *Inst = cast<Instruction>(U);
    IRBuilder<> Builder(Inst);
    Value *ThreadID = Builder.CreateIntrinsic(Intrinsic::xcore_getid, {});
    Value *Addr = Builder.CreateInBoundsGEP(NewGV->getValueType(), NewGV,
                                            {Builder.getInt64(0), ThreadID});
    U->replaceUsesOfWith(GV, Addr);
  }
```
- **EN**: Implements logic around `getType`, `isExternallyInitialized`, `Users`, `cast<Instruction>`, ....
- **CN**: 围绕 `getType`, `isExternallyInitialized`, `Users`, `cast<Instruction>`, ... 实现具体逻辑。

### Lines 162-168
```cpp

  // Remove old global.
  NewGV->takeName(GV);
  GV->eraseFromParent();
  return true;
}

```
- **EN**: Implements logic around `takeName`, `eraseFromParent`; this block returns target-specific results.
- **CN**: 围绕 `takeName`, `eraseFromParent` 实现具体逻辑；这一段返回目标相关结果。

### Lines 169-179
```cpp
bool XCoreLowerThreadLocal::runOnModule(Module &M) {
  // Find thread local globals.
  bool MadeChange = false;
  SmallVector<GlobalVariable *, 16> ThreadLocalGlobals;
  for (GlobalVariable &GV : M.globals())
    if (GV.isThreadLocal())
      ThreadLocalGlobals.push_back(&GV);
  for (GlobalVariable *GV : ThreadLocalGlobals)
    MadeChange |= lowerGlobal(GV);
  return MadeChange;
}
```
- **EN**: Implements logic around `runOnModule`, `push_back`, `lowerGlobal`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `runOnModule`, `push_back`, `lowerGlobal` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

## Key Concepts / 关键概念

- **Thread-local storage / 线程本地存储**:
  - **EN**: Lowers TLS references according to the target ABI
  - **CN**: 按目标 ABI 降低 TLS 引用

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `XCore.h`, `llvm/IR/Constants.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/GlobalVariable.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Intrinsics.h`, `llvm/IR/IntrinsicsXCore.h`, `llvm/IR/Module.h`, `llvm/IR/ValueHandle.h`, `llvm/Pass.h`, `llvm/Support/CommandLine.h`, `llvm/Transforms/Utils/BasicBlockUtils.h`
- **LLVM subsystems / LLVM 子系统**: IR, Support
