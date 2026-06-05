# RandomIRBuilder.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/FuzzMutate/RandomIRBuilder.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements IR mutation utilities used for fuzzing and randomized testing.
  - **CN**: 实现用于模糊测试与随机化测试的 IR 变异工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```cpp
//===-- RandomIRBuilder.cpp -----------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/FuzzMutate/RandomIRBuilder.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/FuzzMutate/OpDescriptor.h"
#include "llvm/FuzzMutate/Random.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Module.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/FuzzMutate/RandomIRBuilder.h`, `llvm/ADT/STLExtras.h`, `llvm/FuzzMutate/OpDescriptor.h`, `llvm/FuzzMutate/Random.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/FuzzMutate/RandomIRBuilder.h`, `llvm/ADT/STLExtras.h`, `llvm/FuzzMutate/OpDescriptor.h`, `llvm/FuzzMutate/Random.h`。

### Lines 21-35
```cpp
using namespace llvm;
using namespace fuzzerop;

static DominatorTree getDomTree(Function &F) {
  // Dominator tree construction requires that all blocks have terminators.
  SmallVector<Instruction *> AddedInsts;
  for (BasicBlock &BB : F)
    if (!BB.hasTerminator())
      AddedInsts.push_back(new UnreachableInst(F.getContext(), &BB));
  DominatorTree DT(F);
  for (Instruction *I : AddedInsts)
    I->eraseFromParent();
  return DT;
}

```
- **EN**: Introduces declarations for `llvm`, `fuzzerop`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `llvm`, `fuzzerop` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 36-54
```cpp
/// Return a vector of Blocks that dominates this block, excluding current
/// block.
static std::vector<BasicBlock *> getDominators(BasicBlock *BB) {
  std::vector<BasicBlock *> ret;
  DominatorTree DT = getDomTree(*BB->getParent());
  DomTreeNode *Node = DT.getNode(BB);
  // It's possible that an orphan block is not in the dom tree. In that case we
  // just return nothing.
  if (!Node)
    return ret;
  Node = Node->getIDom();
  while (Node && Node->getBlock()) {
    ret.push_back(Node->getBlock());
    // Get parent block.
    Node = Node->getIDom();
  }
  return ret;
}

```
- **EN**: Implements logic around `getDominators`, `getDomTree`, `getNode`, `getIDom`, and 2 more symbols.
- **CN**: 围绕 `getDominators`, `getDomTree`, `getNode`, `getIDom`, and 2 more symbols 实现具体逻辑。

### Lines 55-74
```cpp
/// Return a vector of Blocks that is dominated by this block, excluding current
/// block
static std::vector<BasicBlock *> getDominatees(BasicBlock *BB) {
  DominatorTree DT = getDomTree(*BB->getParent());
  std::vector<BasicBlock *> ret;
  DomTreeNode *Parent = DT.getNode(BB);
  // It's possible that an orphan block is not in the dom tree. In that case we
  // just return nothing.
  if (!Parent)
    return ret;
  for (DomTreeNode *Child : Parent->children())
    ret.push_back(Child->getBlock());
  uint64_t Idx = 0;
  while (Idx < ret.size()) {
    DomTreeNode *Node = DT[ret[Idx]];
    Idx++;
    for (DomTreeNode *Child : Node->children())
      ret.push_back(Child->getBlock());
  }
  return ret;
```
- **EN**: Implements logic around `getDominatees`, `getDomTree`, `getNode`, `children`, and 2 more symbols.
- **CN**: 围绕 `getDominatees`, `getDomTree`, `getNode`, `children`, and 2 more symbols 实现具体逻辑。

### Lines 75-88
```cpp
}

AllocaInst *RandomIRBuilder::createStackMemory(Function *F, Type *Ty,
                                               Value *Init) {
  /// TODO: For all Allocas, maybe allocate an array.
  BasicBlock *EntryBB = &F->getEntryBlock();
  const DataLayout &DL = F->getDataLayout();
  AllocaInst *Alloca = new AllocaInst(Ty, DL.getAllocaAddrSpace(), "A",
                                      EntryBB->getFirstInsertionPt());
  if (Init)
    new StoreInst(Init, Alloca, std::next(Alloca->getIterator()));
  return Alloca;
}

```
- **EN**: Implements logic around `createStackMemory`, `getEntryBlock`, `getDataLayout`, `AllocaInst`, and 2 more symbols.
- **CN**: 围绕 `createStackMemory`, `getEntryBlock`, `getDataLayout`, `AllocaInst`, and 2 more symbols 实现具体逻辑。

### Lines 89-108
```cpp
std::pair<GlobalVariable *, bool>
RandomIRBuilder::findOrCreateGlobalVariable(Module *M, ArrayRef<Value *> Srcs,
                                            fuzzerop::SourcePred Pred) {
  auto MatchesPred = [&Srcs, &Pred](GlobalVariable *GV) {
    // Can't directly compare GV's type, as it would be a pointer to the actual
    // type.
    return Pred.matches(Srcs, PoisonValue::get(GV->getValueType()));
  };
  bool DidCreate = false;
  SmallVector<GlobalVariable *, 4> GlobalVars(
      llvm::make_pointer_range(M->globals()));
  auto RS = makeSampler(Rand, make_filter_range(GlobalVars, MatchesPred));
  RS.sample(nullptr, 1);
  GlobalVariable *GV = RS.getSelection();
  if (!GV) {
    DidCreate = true;
    using LinkageTypes = GlobalVariable::LinkageTypes;
    auto TRS = makeSampler<Constant *>(Rand);
    TRS.sample(Pred.generate(Srcs, KnownTypes));
    Constant *Init = TRS.getSelection();
```
- **EN**: Implements logic around `findOrCreateGlobalVariable`, `matches`, `GlobalVars`, `make_pointer_range`, and 3 more symbols.
- **CN**: 围绕 `findOrCreateGlobalVariable`, `matches`, `GlobalVars`, `make_pointer_range`, and 3 more symbols 实现具体逻辑。

### Lines 109-122
```cpp
    Type *Ty = Init->getType();
    GV = new GlobalVariable(*M, Ty, false, LinkageTypes::ExternalLinkage, Init,
                            "G", nullptr,
                            GlobalValue::ThreadLocalMode::NotThreadLocal,
                            M->getDataLayout().getDefaultGlobalsAddressSpace());
  }
  return {GV, DidCreate};
}

Value *RandomIRBuilder::findOrCreateSource(BasicBlock &BB,
                                           ArrayRef<Instruction *> Insts) {
  return findOrCreateSource(BB, Insts, {}, anyType());
}

```
- **EN**: Implements logic around `getType`, `GlobalVariable`, `getDataLayout`, `findOrCreateSource`.
- **CN**: 围绕 `getType`, `GlobalVariable`, `getDataLayout`, `findOrCreateSource` 实现具体逻辑。

### Lines 123-139
```cpp
// Adapts the current pointer for a legal mem operation on the target arch.
static Value *buildTargetLegalPtr(Module *M, Value *Ptr, InsertPosition IP,
                                  const Twine &Name,
                                  SmallVector<Instruction *> *NewInsts) {
  if (M && M->getTargetTriple().isAMDGCN()) {
    // Check if we should perform an address space cast
    PointerType *pointerType = dyn_cast<PointerType>(Ptr->getType());
    if (pointerType && pointerType->getAddressSpace() == 8) {
      // Perform address space cast from address space 8 to address space 7
      auto NewPtr = new AddrSpaceCastInst(
          Ptr, PointerType::get(M->getContext(), 7), Name + ".ASC", IP);
      if (NewInsts)
        NewInsts->push_back(NewPtr);
      return NewPtr;
    }
  }

```
- **EN**: Implements logic around `buildTargetLegalPtr`, `getTargetTriple`, `dyn_cast`, `getAddressSpace`, and 3 more symbols.
- **CN**: 围绕 `buildTargetLegalPtr`, `getTargetTriple`, `dyn_cast`, `getAddressSpace`, and 3 more symbols 实现具体逻辑。

### Lines 140-150
```cpp
  return Ptr;
}

// Stores a value to memory, considering the target triple's restrictions.
static Instruction *buildTargetLegalStore(Value *Val, Value *Ptr,
                                          InsertPosition IP, Module *M) {
  Value *StorePtr = buildTargetLegalPtr(M, Ptr, IP, "", nullptr);
  Instruction *Store = new StoreInst(Val, StorePtr, IP);
  return Store;
}

```
- **EN**: Implements logic around `buildTargetLegalStore`, `buildTargetLegalPtr`, `StoreInst`.
- **CN**: 围绕 `buildTargetLegalStore`, `buildTargetLegalPtr`, `StoreInst` 实现具体逻辑。

### Lines 151-161
```cpp
// Loads a value from memory, considering the target triple's restrictions.
static std::pair<Instruction *, SmallVector<Instruction *>>
buildTargetLegalLoad(Type *AccessTy, Value *Ptr, InsertPosition IP, Module *M,
                     const Twine &LoadName) {
  SmallVector<Instruction *> NewInsts;

  Value *LoadPtr = buildTargetLegalPtr(M, Ptr, IP, LoadName, &NewInsts);

  Instruction *Load = new LoadInst(AccessTy, LoadPtr, LoadName, IP);
  NewInsts.push_back(Load);

```
- **EN**: Implements logic around `buildTargetLegalLoad`, `buildTargetLegalPtr`, `LoadInst`, `push_back`.
- **CN**: 围绕 `buildTargetLegalLoad`, `buildTargetLegalPtr`, `LoadInst`, `push_back` 实现具体逻辑。

### Lines 162-171
```cpp
  return std::make_pair(Load, NewInsts);
}

static void eraseNewInstructions(SmallVector<Instruction *> &NewInsts) {
  // Remove in reverse order (uses before defs)
  for (auto it = NewInsts.rbegin(); it != NewInsts.rend(); ++it) {
    (*it)->eraseFromParent();
  }
}

```
- **EN**: Implements logic around `make_pair`, `eraseNewInstructions`, `rbegin`, `eraseFromParent`.
- **CN**: 围绕 `make_pair`, `eraseNewInstructions`, `rbegin`, `eraseFromParent` 实现具体逻辑。

### Lines 172-191
```cpp
Value *RandomIRBuilder::findOrCreateSource(BasicBlock &BB,
                                           ArrayRef<Instruction *> Insts,
                                           ArrayRef<Value *> Srcs,
                                           SourcePred Pred,
                                           bool allowConstant) {
  auto MatchesPred = [&Srcs, &Pred](Value *V) { return Pred.matches(Srcs, V); };
  SmallVector<uint64_t, 8> SrcTys;
  for (uint64_t i = 0; i < EndOfValueSource; i++)
    SrcTys.push_back(i);
  std::shuffle(SrcTys.begin(), SrcTys.end(), Rand);
  for (uint64_t SrcTy : SrcTys) {
    switch (SrcTy) {
    case SrcFromInstInCurBlock: {
      auto RS = makeSampler(Rand, make_filter_range(Insts, MatchesPred));
      if (!RS.isEmpty()) {
        return RS.getSelection();
      }
      break;
    }
    case FunctionArgument: {
```
- **EN**: Implements logic around `findOrCreateSource`, `matches`, `push_back`, `shuffle`, and 3 more symbols; this block uses `switch`-style dispatch.
- **CN**: 围绕 `findOrCreateSource`, `matches`, `push_back`, `shuffle`, and 3 more symbols 实现具体逻辑；该代码块使用 `switch` 风格分派。

### Lines 192-211
```cpp
      Function *F = BB.getParent();
      SmallVector<Argument *, 8> Args;
      for (uint64_t i = 0; i < F->arg_size(); i++) {
        Args.push_back(F->getArg(i));
      }
      auto RS = makeSampler(Rand, make_filter_range(Args, MatchesPred));
      if (!RS.isEmpty()) {
        return RS.getSelection();
      }
      break;
    }
    case InstInDominator: {
      auto Dominators = getDominators(&BB);
      std::shuffle(Dominators.begin(), Dominators.end(), Rand);
      for (BasicBlock *Dom : Dominators) {
        SmallVector<Instruction *, 16> Instructions(
            llvm::make_pointer_range(*Dom));
        auto RS =
            makeSampler(Rand, make_filter_range(Instructions, MatchesPred));
        // Also consider choosing no source, meaning we want a new one.
```
- **EN**: Implements logic around `getParent`, `arg_size`, `push_back`, `makeSampler`, and 6 more symbols.
- **CN**: 围绕 `getParent`, `arg_size`, `push_back`, `makeSampler`, and 6 more symbols 实现具体逻辑。

### Lines 212-231
```cpp
        if (!RS.isEmpty()) {
          return RS.getSelection();
        }
      }
      break;
    }
    case SrcFromGlobalVariable: {
      Module *M = BB.getParent()->getParent();
      auto [GV, DidCreate] = findOrCreateGlobalVariable(M, Srcs, Pred);
      Type *Ty = GV->getValueType();
      InsertPosition IP = BB.hasTerminator()
                              ? InsertPosition(BB.getFirstInsertionPt())
                              : InsertPosition(&BB);
      // Build a legal load and track new instructions in case a rollback is
      // needed.
      auto [LoadGV, NewInsts] = buildTargetLegalLoad(Ty, GV, IP, M, "LGV");
      // Because we might be generating new values, we have to check if it
      // matches again.
      if (DidCreate) {
        if (Pred.matches(Srcs, LoadGV)) {
```
- **EN**: Implements logic around `isEmpty`, `getSelection`, `getParent`, `findOrCreateGlobalVariable`, and 5 more symbols.
- **CN**: 围绕 `isEmpty`, `getSelection`, `getParent`, `findOrCreateGlobalVariable`, and 5 more symbols 实现具体逻辑。

### Lines 232-251
```cpp
          return LoadGV;
        }
        // Remove newly inserted instructions
        eraseNewInstructions(NewInsts);
        // If no one is using this GlobalVariable, delete it too.
        if (GV->use_empty()) {
          GV->eraseFromParent();
        }
      }
      break;
    }
    case NewConstOrStack: {
      return newSource(BB, Insts, Srcs, Pred, allowConstant);
    }
    default:
    case EndOfValueSource: {
      llvm_unreachable("EndOfValueSource executed");
    }
    }
  }
```
- **EN**: Implements logic around `eraseNewInstructions`, `use_empty`, `eraseFromParent`, `newSource`, and 1 more symbols.
- **CN**: 围绕 `eraseNewInstructions`, `use_empty`, `eraseFromParent`, `newSource`, and 1 more symbols 实现具体逻辑。

### Lines 252-261
```cpp
  llvm_unreachable("Can't find a source");
}

Value *RandomIRBuilder::newSource(BasicBlock &BB, ArrayRef<Instruction *> Insts,
                                  ArrayRef<Value *> Srcs, SourcePred Pred,
                                  bool allowConstant) {
  // Generate some constants to choose from.
  auto RS = makeSampler<Value *>(Rand);
  RS.sample(Pred.generate(Srcs, KnownTypes));

```
- **EN**: Implements logic around `llvm_unreachable`, `newSource`, `sample`.
- **CN**: 围绕 `llvm_unreachable`, `newSource`, `sample` 实现具体逻辑。

### Lines 262-277
```cpp
  // If we can find a pointer to load from, use it half the time.
  Value *Ptr = findPointer(BB, Insts);
  if (Ptr) {
    // Create load from the chosen pointer
    auto IP = BB.getFirstInsertionPt();
    if (auto *I = dyn_cast<Instruction>(Ptr)) {
      IP = ++I->getIterator();
      assert(IP != BB.end() && "guaranteed by the findPointer");
    }
    // Pick the type independently.
    Type *AccessTy = RS.getSelection()->getType();
    // Build a legal load and track new instructions in case a rollback is
    // needed.
    auto [NewLoad, NewInsts] =
        buildTargetLegalLoad(AccessTy, Ptr, IP, BB.getModule(), "L");

```
- **EN**: Implements logic around `findPointer`, `getFirstInsertionPt`, `dyn_cast`, `getIterator`, and 3 more symbols.
- **CN**: 围绕 `findPointer`, `getFirstInsertionPt`, `dyn_cast`, `getIterator`, and 3 more symbols 实现具体逻辑。

### Lines 278-297
```cpp
    // Only sample this load if it really matches the descriptor
    if (Pred.matches(Srcs, NewLoad))
      RS.sample(NewLoad, RS.totalWeight());
    else {
      // Remove newly inserted instructions
      eraseNewInstructions(NewInsts);
    }
  }

  Value *newSrc = RS.getSelection();
  // Generate a stack alloca and store the constant to it if constant is not
  // allowed, our hope is that later mutations can generate some values and
  // store to this placeholder.
  if (!allowConstant && isa<Constant>(newSrc)) {
    Type *Ty = newSrc->getType();
    Function *F = BB.getParent();
    AllocaInst *Alloca = createStackMemory(F, Ty, newSrc);
    if (BB.hasTerminator()) {
      newSrc = new LoadInst(Ty, Alloca, /*ArrLen,*/ "L",
                            BB.getTerminator()->getIterator());
```
- **EN**: Implements logic around `matches`, `sample`, `eraseNewInstructions`, `getSelection`, and 7 more symbols.
- **CN**: 围绕 `matches`, `sample`, `eraseNewInstructions`, `getSelection`, and 7 more symbols 实现具体逻辑。

### Lines 298-317
```cpp
    } else {
      newSrc = new LoadInst(Ty, Alloca, /*ArrLen,*/ "L", &BB);
    }
  }
  return newSrc;
}

static bool isCompatibleReplacement(const Instruction *I, const Use &Operand,
                                    const Value *Replacement) {
  unsigned int OperandNo = Operand.getOperandNo();
  if (Operand->getType() != Replacement->getType())
    return false;
  switch (I->getOpcode()) {
  case Instruction::GetElementPtr:
  case Instruction::ExtractElement:
  case Instruction::ExtractValue:
    // TODO: We could potentially validate these, but for now just leave indices
    // alone.
    if (OperandNo >= 1)
      return false;
```
- **EN**: Implements logic around `LoadInst`, `isCompatibleReplacement`, `getOperandNo`, `getType`, and 1 more symbols; this block uses `switch`-style dispatch.
- **CN**: 围绕 `LoadInst`, `isCompatibleReplacement`, `getOperandNo`, `getType`, and 1 more symbols 实现具体逻辑；该代码块使用 `switch` 风格分派。

### Lines 318-337
```cpp
    break;
  case Instruction::InsertValue:
  case Instruction::InsertElement:
  case Instruction::ShuffleVector:
    if (OperandNo >= 2)
      return false;
    break;
  // For Br/Switch, we only try to modify the 1st Operand (condition).
  // Modify other operands, like switch case may accidently change case from
  // ConstantInt to a register, which is illegal.
  case Instruction::Switch:
  case Instruction::CondBr:
    if (OperandNo >= 1)
      return false;
    break;
  case Instruction::Call:
  case Instruction::Invoke:
  case Instruction::CallBr: {
    const Function *Callee = cast<CallBase>(I)->getCalledFunction();
    // If it's an indirect call, give up.
```
- **EN**: Implements logic around `cast`; this block uses `switch`-style dispatch.
- **CN**: 围绕 `cast` 实现具体逻辑；该代码块使用 `switch` 风格分派。

### Lines 338-357
```cpp
    if (!Callee)
      return false;
    // If callee is not an intrinsic, operand 0 is the function to be called.
    // Since we cannot assume that the replacement is a function pointer,
    // we give up.
    if (!Callee->getIntrinsicID() && OperandNo == 0)
      return false;
    return !Callee->hasParamAttribute(OperandNo, Attribute::ImmArg);
  }
  case Instruction::CatchPad:
    // Argument operand must be alloca or constant
    if (!isa<Constant>(Replacement) && !isa<AllocaInst>(Replacement))
      return false;
    break;
  default:
    break;
  }
  return true;
}

```
- **EN**: Implements logic around `getIntrinsicID`, `hasParamAttribute`, `isa`.
- **CN**: 围绕 `getIntrinsicID`, `hasParamAttribute`, `isa` 实现具体逻辑。

### Lines 358-377
```cpp
Instruction *RandomIRBuilder::connectToSink(BasicBlock &BB,
                                            ArrayRef<Instruction *> Insts,
                                            Value *V) {
  SmallVector<uint64_t, 8> SinkTys;
  for (uint64_t i = 0; i < EndOfValueSink; i++)
    SinkTys.push_back(i);
  std::shuffle(SinkTys.begin(), SinkTys.end(), Rand);
  auto findSinkAndConnect =
      [this, V](ArrayRef<Instruction *> Instructions) -> Instruction * {
    auto RS = makeSampler<Use *>(Rand);
    for (auto &I : Instructions) {
      for (Use &U : I->operands())
        if (isCompatibleReplacement(I, U, V))
          RS.sample(&U, 1);
    }
    if (!RS.isEmpty()) {
      Use *Sink = RS.getSelection();
      User *U = Sink->getUser();
      unsigned OpNo = Sink->getOperandNo();
      U->setOperand(OpNo, V);
```
- **EN**: Implements logic around `connectToSink`, `push_back`, `shuffle`, `operands`, and 7 more symbols.
- **CN**: 围绕 `connectToSink`, `push_back`, `shuffle`, `operands`, and 7 more symbols 实现具体逻辑。

### Lines 378-397
```cpp
      return cast<Instruction>(U);
    }
    return nullptr;
  };
  Instruction *Sink = nullptr;
  for (uint64_t SinkTy : SinkTys) {
    switch (SinkTy) {
    case SinkToInstInCurBlock:
      Sink = findSinkAndConnect(Insts);
      if (Sink)
        return Sink;
      break;
    case PointersInDominator: {
      auto Dominators = getDominators(&BB);
      std::shuffle(Dominators.begin(), Dominators.end(), Rand);
      for (BasicBlock *Dom : Dominators) {
        for (Instruction &I : *Dom) {
          if (isa<PointerType>(I.getType())) {
            return buildTargetLegalStore(V, &I, Insts.back()->getIterator(),
                                         I.getModule());
```
- **EN**: Implements logic around `cast`, `findSinkAndConnect`, `getDominators`, `shuffle`, and 3 more symbols; this block uses `switch`-style dispatch.
- **CN**: 围绕 `cast`, `findSinkAndConnect`, `getDominators`, `shuffle`, and 3 more symbols 实现具体逻辑；该代码块使用 `switch` 风格分派。

### Lines 398-417
```cpp
          }
        }
      }
      break;
    }
    case InstInDominatee: {
      auto Dominatees = getDominatees(&BB);
      std::shuffle(Dominatees.begin(), Dominatees.end(), Rand);
      for (BasicBlock *Dominee : Dominatees) {
        std::vector<Instruction *> Instructions;
        for (Instruction &I : *Dominee)
          Instructions.push_back(&I);
        Sink = findSinkAndConnect(Instructions);
        if (Sink) {
          return Sink;
        }
      }
      break;
    }
    case NewStore:
```
- **EN**: Implements logic around `getDominatees`, `shuffle`, `push_back`, `findSinkAndConnect`.
- **CN**: 围绕 `getDominatees`, `shuffle`, `push_back`, `findSinkAndConnect` 实现具体逻辑。

### Lines 418-433
```cpp
      /// TODO: allocate a new stack memory.
      return newSink(BB, Insts, V);
    case SinkToGlobalVariable: {
      Module *M = BB.getModule();
      auto [GV, DidCreate] =
          findOrCreateGlobalVariable(M, {}, fuzzerop::onlyType(V->getType()));
      return buildTargetLegalStore(V, GV, Insts.back()->getIterator(), M);
    }
    case EndOfValueSink:
    default:
      llvm_unreachable("EndOfValueSink executed");
    }
  }
  llvm_unreachable("Can't find a sink");
}

```
- **EN**: Implements logic around `newSink`, `getModule`, `findOrCreateGlobalVariable`, `buildTargetLegalStore`, and 1 more symbols.
- **CN**: 围绕 `newSink`, `getModule`, `findOrCreateGlobalVariable`, `buildTargetLegalStore`, and 1 more symbols 实现具体逻辑。

### Lines 434-445
```cpp
Instruction *RandomIRBuilder::newSink(BasicBlock &BB,
                                      ArrayRef<Instruction *> Insts, Value *V) {
  Value *Ptr = findPointer(BB, Insts);
  if (!Ptr) {
    if (uniform(Rand, 0, 1)) {
      Type *Ty = V->getType();
      Ptr = createStackMemory(BB.getParent(), Ty, PoisonValue::get(Ty));
    } else {
      Ptr = PoisonValue::get(PointerType::get(V->getContext(), 0));
    }
  }

```
- **EN**: Implements logic around `newSink`, `findPointer`, `uniform`, `getType`, and 2 more symbols.
- **CN**: 围绕 `newSink`, `findPointer`, `uniform`, `getType`, and 2 more symbols 实现具体逻辑。

### Lines 446-457
```cpp
  return buildTargetLegalStore(V, Ptr, Insts.back()->getIterator(),
                               BB.getModule());
}

Value *RandomIRBuilder::findPointer(BasicBlock &BB,
                                    ArrayRef<Instruction *> Insts) {
  auto IsMatchingPtr = [](Instruction *Inst) {
    // Invoke instructions sometimes produce valid pointers but currently
    // we can't insert loads or stores from them
    if (Inst->isTerminator())
      return false;

```
- **EN**: Implements logic around `buildTargetLegalStore`, `getModule`, `findPointer`, `isTerminator`.
- **CN**: 围绕 `buildTargetLegalStore`, `getModule`, `findPointer`, `isTerminator` 实现具体逻辑。

### Lines 458-469
```cpp
    return Inst->getType()->isPointerTy();
  };
  if (auto RS = makeSampler(Rand, make_filter_range(Insts, IsMatchingPtr)))
    return RS.getSelection();
  return nullptr;
}

Type *RandomIRBuilder::randomType() {
  uint64_t TyIdx = uniform<uint64_t>(Rand, 0, KnownTypes.size() - 1);
  return KnownTypes[TyIdx];
}

```
- **EN**: Implements logic around `getType`, `makeSampler`, `getSelection`, `randomType`, and 1 more symbols.
- **CN**: 围绕 `getType`, `makeSampler`, `getSelection`, `randomType`, and 1 more symbols 实现具体逻辑。

### Lines 470-488
```cpp
Function *RandomIRBuilder::createFunctionDeclaration(Module &M,
                                                     uint64_t ArgNum) {
  Type *RetType = randomType();

  SmallVector<Type *, 2> Args;
  for (uint64_t i = 0; i < ArgNum; i++) {
    Args.push_back(randomType());
  }

  Function *F = Function::Create(FunctionType::get(RetType, Args,
                                                   /*isVarArg=*/false),
                                 GlobalValue::ExternalLinkage, "f", &M);
  return F;
}
Function *RandomIRBuilder::createFunctionDeclaration(Module &M) {
  return createFunctionDeclaration(
      M, uniform<uint64_t>(Rand, MinArgNum, MaxArgNum));
}

```
- **EN**: Implements logic around `createFunctionDeclaration`, `randomType`, `push_back`, `Create`, and 1 more symbols.
- **CN**: 围绕 `createFunctionDeclaration`, `randomType`, `push_back`, `Create`, and 1 more symbols 实现具体逻辑。

### Lines 489-507
```cpp
Function *RandomIRBuilder::createFunctionDefinition(Module &M,
                                                    uint64_t ArgNum) {
  Function *F = this->createFunctionDeclaration(M, ArgNum);

  // TODO: Some arguments and a return value would probably be more
  // interesting.
  LLVMContext &Context = M.getContext();
  const DataLayout &DL = M.getDataLayout();
  BasicBlock *BB = BasicBlock::Create(Context, "BB", F);
  Type *RetTy = F->getReturnType();
  if (RetTy != Type::getVoidTy(Context)) {
    Instruction *RetAlloca =
        new AllocaInst(RetTy, DL.getAllocaAddrSpace(), "RP", BB);
    Instruction *RetLoad = new LoadInst(RetTy, RetAlloca, "", BB);
    ReturnInst::Create(Context, RetLoad, BB);
  } else {
    ReturnInst::Create(Context, BB);
  }

```
- **EN**: Implements logic around `createFunctionDefinition`, `createFunctionDeclaration`, `getContext`, `getDataLayout`, and 5 more symbols.
- **CN**: 围绕 `createFunctionDefinition`, `createFunctionDeclaration`, `getContext`, `getDataLayout`, and 5 more symbols 实现具体逻辑。

### Lines 508-513
```cpp
  return F;
}
Function *RandomIRBuilder::createFunctionDefinition(Module &M) {
  return createFunctionDefinition(
      M, uniform<uint64_t>(Rand, MinArgNum, MaxArgNum));
}
```
- **EN**: Implements logic around `createFunctionDefinition`, `uniform`.
- **CN**: 围绕 `createFunctionDefinition`, `uniform` 实现具体逻辑。

## Key Concepts / 关键概念

- **IR mutation / IR 变异**:
  - **EN**: Builds randomized but valid IR edits for fuzzing workflows.
  - **CN**: 为模糊测试流程构造随机但有效的 IR 变换。
- **LLVM container usage / LLVM 容器使用**:
  - **EN**: Relies on LLVM ADT containers for performance-conscious in-memory data management.
  - **CN**: 依赖 LLVM ADT 容器来进行注重性能的内存数据管理。
- **Concurrency or parallel work / 并发或并行工作**:
  - **EN**: Coordinates tasks that may execute concurrently or partition work.
  - **CN**: 协调可能并发执行或分片处理的任务。

## Dependencies / 依赖关系

- **Direct LLVM/local includes / 直接的 LLVM/本地包含**: `llvm/FuzzMutate/RandomIRBuilder.h`, `llvm/ADT/STLExtras.h`, `llvm/FuzzMutate/OpDescriptor.h`, `llvm/FuzzMutate/Random.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/Instructions.h` ... (+1 more)
- **Subsystem categories / 子系统类别**: LLVM IR core abstractions / LLVM IR 核心抽象 (7), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1)
