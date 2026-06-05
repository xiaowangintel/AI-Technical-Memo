# AMDGPURewriteOutArguments.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPURewriteOutArguments.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPURewriteOutArguments for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPURewriteOutArguments 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: File banner, license, and overview
```cpp
//===- AMDGPURewriteOutArgumentsPass.cpp - Create struct returns ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file This pass attempts to replace out argument usage with a return of a
/// struct.
///
/// We can support returning a lot of values directly in registers, but
/// idiomatic C code frequently uses a pointer argument to return a second value
/// rather than returning a struct by value. GPU stack access is also quite
/// painful, so we want to avoid that if possible. Passing a stack object
/// pointer to a function also requires an additional address expansion code
/// sequence to convert the pointer to be relative to the kernel's scratch wave
/// offset register since the callee doesn't know what stack frame the incoming
/// pointer is relative to.
///
/// The goal is to try rewriting code that looks like this:
///
///  int foo(int a, int b, int* out) {
///     *out = bar();
///     return a + b;
/// }
///
/// into something like this:
///
///  std::pair<int, int> foo(int a, int b) {
///     return std::pair(a + b, bar());
/// }
///
/// Typically the incoming pointer is a simple alloca for a temporary variable
```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins. Main symbols: `std::pair`.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。 主要符号：`std::pair`。

### Lines 35-67: Header dependencies and setup
```cpp
/// to use the API, which if replaced with a struct return will be easily SROA'd
/// out when the stub function we create is inlined
///
/// This pass introduces the struct return, but leaves the unused pointer
/// arguments and introduces a new stub function calling the struct returning
/// body. DeadArgumentElimination should be run after this to clean these up.
//
//===----------------------------------------------------------------------===//

#include "AMDGPU.h"
#include "Utils/AMDGPUBaseInfo.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/Analysis/MemoryDependenceAnalysis.h"
#include "llvm/IR/AttributeMask.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/Instructions.h"
#include "llvm/InitializePasses.h"
#include "llvm/Pass.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"

#define DEBUG_TYPE "amdgpu-rewrite-out-arguments"

using namespace llvm;

static cl::opt<bool> AnyAddressSpace(
  "amdgpu-any-address-space-out-arguments",
  cl::desc("Replace pointer out arguments with "
           "struct returns for non-private address space"),
  cl::Hidden,
  cl::init(false));

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `cl::desc`, `cl::init`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`cl::desc`, `cl::init`。

### Lines 68-98: Declares class AMDGPURewriteOutArguments
```cpp
static cl::opt<unsigned> MaxNumRetRegs(
  "amdgpu-max-return-arg-num-regs",
  cl::desc("Approximately limit number of return registers for replacing out arguments"),
  cl::Hidden,
  cl::init(16));

STATISTIC(NumOutArgumentsReplaced,
          "Number out arguments moved to struct return values");
STATISTIC(NumOutArgumentFunctionsReplaced,
          "Number of functions with out arguments moved to struct return values");

namespace {

class AMDGPURewriteOutArguments : public FunctionPass {
private:
  const DataLayout *DL = nullptr;
  MemoryDependenceResults *MDA = nullptr;

  Type *getStoredType(Value &Arg) const;
  Type *getOutArgumentType(Argument &Arg) const;

public:
  static char ID;

  AMDGPURewriteOutArguments() : FunctionPass(ID) {}

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<MemoryDependenceWrapperPass>();
    FunctionPass::getAnalysisUsage(AU);
  }

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPURewriteOutArguments`, `cl::desc`, `cl::init`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPURewriteOutArguments`, `cl::desc`, `cl::init`。

### Lines 99-132: Registers LLVM passes
```cpp
  bool doInitialization(Module &M) override;
  bool runOnFunction(Function &F) override;
};

} // end anonymous namespace

INITIALIZE_PASS_BEGIN(AMDGPURewriteOutArguments, DEBUG_TYPE,
                      "AMDGPU Rewrite Out Arguments", false, false)
INITIALIZE_PASS_DEPENDENCY(MemoryDependenceWrapperPass)
INITIALIZE_PASS_END(AMDGPURewriteOutArguments, DEBUG_TYPE,
                    "AMDGPU Rewrite Out Arguments", false, false)

char AMDGPURewriteOutArguments::ID = 0;

Type *AMDGPURewriteOutArguments::getStoredType(Value &Arg) const {
  const int MaxUses = 10;
  int UseCount = 0;

  SmallVector<Use *> Worklist(llvm::make_pointer_range(Arg.uses()));

  Type *StoredType = nullptr;
  while (!Worklist.empty()) {
    Use *U = Worklist.pop_back_val();

    if (auto *BCI = dyn_cast<BitCastInst>(U->getUser())) {
      for (Use &U : BCI->uses())
        Worklist.push_back(&U);
      continue;
    }

    if (auto *SI = dyn_cast<StoreInst>(U->getUser())) {
      if (UseCount++ > MaxUses)
        return nullptr;

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `AMDGPURewriteOutArguments::getStoredType`, `llvm::make_pointer_range`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`AMDGPURewriteOutArguments::getStoredType`, `llvm::make_pointer_range`。

### Lines 133-164: Conditional logic and checks
```cpp
      if (!SI->isSimple() ||
          U->getOperandNo() != StoreInst::getPointerOperandIndex())
        return nullptr;

      if (StoredType && StoredType != SI->getValueOperand()->getType())
        return nullptr; // More than one type.
      StoredType = SI->getValueOperand()->getType();
      continue;
    }

    // Unsupported user.
    return nullptr;
  }

  return StoredType;
}

Type *AMDGPURewriteOutArguments::getOutArgumentType(Argument &Arg) const {
  const unsigned MaxOutArgSizeBytes = 4 * MaxNumRetRegs;
  PointerType *ArgTy = dyn_cast<PointerType>(Arg.getType());

  // TODO: It might be useful for any out arguments, not just privates.
  if (!ArgTy || (ArgTy->getAddressSpace() != DL->getAllocaAddrSpace() &&
                 !AnyAddressSpace) ||
      Arg.hasByValAttr() || Arg.hasStructRetAttr()) {
    return nullptr;
  }

  Type *StoredType = getStoredType(Arg);
  if (!StoredType || DL->getTypeStoreSize(StoredType) > MaxOutArgSizeBytes)
    return nullptr;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `StoreInst::getPointerOperandIndex`, `AMDGPURewriteOutArguments::getOutArgumentType`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`StoreInst::getPointerOperandIndex`, `AMDGPURewriteOutArguments::getOutArgumentType`。

### Lines 165-196: Implements AMDGPURewriteOutArguments::doInitialization
```cpp
  return StoredType;
}

bool AMDGPURewriteOutArguments::doInitialization(Module &M) {
  DL = &M.getDataLayout();
  return false;
}

bool AMDGPURewriteOutArguments::runOnFunction(Function &F) {
  if (skipFunction(F))
    return false;

  // TODO: Could probably handle variadic functions.
  if (F.isVarArg() || F.hasStructRetAttr() ||
      AMDGPU::isEntryFunctionCC(F.getCallingConv()))
    return false;

  MDA = &getAnalysis<MemoryDependenceWrapperPass>().getMemDep();

  unsigned ReturnNumRegs = 0;
  SmallDenseMap<int, Type *, 4> OutArgIndexes;
  SmallVector<Type *, 4> ReturnTypes;
  Type *RetTy = F.getReturnType();
  if (!RetTy->isVoidTy()) {
    ReturnNumRegs = DL->getTypeStoreSize(RetTy) / 4;

    if (ReturnNumRegs >= MaxNumRetRegs)
      return false;

    ReturnTypes.push_back(RetTy);
  }

```
**EN:** This section contains concrete logic for AMDGPURewriteOutArguments::doInitialization. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPURewriteOutArguments::doInitialization`, `AMDGPURewriteOutArguments::runOnFunction`, `AMDGPU::isEntryFunctionCC`.
**CN:** 本节包含与 AMDGPURewriteOutArguments::doInitialization 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPURewriteOutArguments::doInitialization`, `AMDGPURewriteOutArguments::runOnFunction`, `AMDGPU::isEntryFunctionCC`。

### Lines 197-226: Type declarations and aliases
```cpp
  SmallVector<std::pair<Argument *, Type *>, 4> OutArgs;
  for (Argument &Arg : F.args()) {
    if (Type *Ty = getOutArgumentType(Arg)) {
      LLVM_DEBUG(dbgs() << "Found possible out argument " << Arg
                        << " in function " << F.getName() << '\n');
      OutArgs.push_back({&Arg, Ty});
    }
  }

  if (OutArgs.empty())
    return false;

  using ReplacementVec = SmallVector<std::pair<Argument *, Value *>, 4>;

  DenseMap<ReturnInst *, ReplacementVec> Replacements;

  SmallVector<ReturnInst *, 4> Returns;
  for (BasicBlock &BB : F) {
    if (ReturnInst *RI = dyn_cast<ReturnInst>(&BB.back()))
      Returns.push_back(RI);
  }

  if (Returns.empty())
    return false;

  bool Changing;

  do {
    Changing = false;

```
**EN:** This section introduces supporting types or aliases that simplify later declarations and backend logic.
**CN:** 本节引入辅助类型或别名，以简化后续声明与后端逻辑。

### Lines 227-260: Conditional logic and checks
```cpp
    // Keep retrying if we are able to successfully eliminate an argument. This
    // helps with cases with multiple arguments which may alias, such as in a
    // sincos implementation. If we have 2 stores to arguments, on the first
    // attempt the MDA query will succeed for the second store but not the
    // first. On the second iteration we've removed that out clobbering argument
    // (by effectively moving it into another function) and will find the second
    // argument is OK to move.
    for (const auto &Pair : OutArgs) {
      bool ThisReplaceable = true;
      SmallVector<std::pair<ReturnInst *, StoreInst *>, 4> ReplaceableStores;

      Argument *OutArg = Pair.first;
      Type *ArgTy = Pair.second;

      // Skip this argument if converting it will push us over the register
      // count to return limit.

      // TODO: This is an approximation. When legalized this could be more. We
      // can ask TLI for exactly how many.
      unsigned ArgNumRegs = DL->getTypeStoreSize(ArgTy) / 4;
      if (ArgNumRegs + ReturnNumRegs > MaxNumRetRegs)
        continue;

      // An argument is convertible only if all exit blocks are able to replace
      // it.
      for (ReturnInst *RI : Returns) {
        BasicBlock *BB = RI->getParent();

        MemDepResult Q = MDA->getPointerDependencyFrom(
            MemoryLocation::getBeforeOrAfter(OutArg), true, BB->end(), BB, RI);
        StoreInst *SI = nullptr;
        if (Q.isDef())
          SI = dyn_cast<StoreInst>(Q.getInst());

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `MemoryLocation::getBeforeOrAfter`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`MemoryLocation::getBeforeOrAfter`。

### Lines 261-289: Conditional logic and checks
```cpp
        if (SI) {
          LLVM_DEBUG(dbgs() << "Found out argument store: " << *SI << '\n');
          ReplaceableStores.emplace_back(RI, SI);
        } else {
          ThisReplaceable = false;
          break;
        }
      }

      if (!ThisReplaceable)
        continue; // Try the next argument candidate.

      for (std::pair<ReturnInst *, StoreInst *> Store : ReplaceableStores) {
        Value *ReplVal = Store.second->getValueOperand();

        auto &ValVec = Replacements[Store.first];
        if (llvm::is_contained(llvm::make_first_range(ValVec), OutArg)) {
          LLVM_DEBUG(dbgs()
                     << "Saw multiple out arg stores" << *OutArg << '\n');
          // It is possible to see stores to the same argument multiple times,
          // but we expect these would have been optimized out already.
          ThisReplaceable = false;
          break;
        }

        ValVec.emplace_back(OutArg, ReplVal);
        Store.second->eraseFromParent();
      }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `llvm::is_contained`, `llvm::make_first_range`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`llvm::is_contained`, `llvm::make_first_range`。

### Lines 290-320: Conditional logic and checks
```cpp
      if (ThisReplaceable) {
        ReturnTypes.push_back(ArgTy);
        OutArgIndexes.insert({OutArg->getArgNo(), ArgTy});
        ++NumOutArgumentsReplaced;
        Changing = true;
      }
    }
  } while (Changing);

  if (Replacements.empty())
    return false;

  LLVMContext &Ctx = F.getContext();
  StructType *NewRetTy = StructType::create(Ctx, ReturnTypes, F.getName());

  FunctionType *NewFuncTy = FunctionType::get(NewRetTy,
                                              F.getFunctionType()->params(),
                                              F.isVarArg());

  LLVM_DEBUG(dbgs() << "Computed new return type: " << *NewRetTy << '\n');

  Function *NewFunc = Function::Create(NewFuncTy, Function::PrivateLinkage,
                                       F.getName() + ".body");
  F.getParent()->getFunctionList().insert(F.getIterator(), NewFunc);
  NewFunc->copyAttributesFrom(&F);
  NewFunc->setComdat(F.getComdat());

  // We want to preserve the function and param attributes, but need to strip
  // off any return attributes, e.g. zeroext doesn't make sense with a struct.
  NewFunc->stealArgumentListFrom(F);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `StructType::create`, `FunctionType::get`, `Function::Create`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`StructType::create`, `FunctionType::get`, `Function::Create`。

### Lines 321-354: Declares addAttribute
```cpp
  AttributeMask RetAttrs;
  RetAttrs.addAttribute(Attribute::SExt);
  RetAttrs.addAttribute(Attribute::ZExt);
  RetAttrs.addAttribute(Attribute::NoAlias);
  NewFunc->removeRetAttrs(RetAttrs);
  // TODO: How to preserve metadata?

  // Move the body of the function into the new rewritten function, and replace
  // this function with a stub.
  NewFunc->splice(NewFunc->begin(), &F);

  for (std::pair<ReturnInst *, ReplacementVec> &Replacement : Replacements) {
    ReturnInst *RI = Replacement.first;
    IRBuilder<> B(RI);
    B.SetCurrentDebugLocation(RI->getDebugLoc());

    int RetIdx = 0;
    Value *NewRetVal = PoisonValue::get(NewRetTy);

    Value *RetVal = RI->getReturnValue();
    if (RetVal)
      NewRetVal = B.CreateInsertValue(NewRetVal, RetVal, RetIdx++);

    for (std::pair<Argument *, Value *> ReturnPoint : Replacement.second)
      NewRetVal = B.CreateInsertValue(NewRetVal, ReturnPoint.second, RetIdx++);

    if (RetVal)
      RI->setOperand(0, NewRetVal);
    else {
      B.CreateRet(NewRetVal);
      RI->eraseFromParent();
    }
  }

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `PoisonValue::get`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`PoisonValue::get`。

### Lines 355-383: Implements BasicBlock::Create
```cpp
  SmallVector<Value *, 16> StubCallArgs;
  for (Argument &Arg : F.args()) {
    if (OutArgIndexes.count(Arg.getArgNo())) {
      // It's easier to preserve the type of the argument list. We rely on
      // DeadArgumentElimination to take care of these.
      StubCallArgs.push_back(PoisonValue::get(Arg.getType()));
    } else {
      StubCallArgs.push_back(&Arg);
    }
  }

  BasicBlock *StubBB = BasicBlock::Create(Ctx, "", &F);
  IRBuilder<> B(StubBB);
  CallInst *StubCall = B.CreateCall(NewFunc, StubCallArgs);

  int RetIdx = RetTy->isVoidTy() ? 0 : 1;
  for (Argument &Arg : F.args()) {
    auto It = OutArgIndexes.find(Arg.getArgNo());
    if (It == OutArgIndexes.end())
      continue;

    Type *EltTy = It->second;
    const auto Align =
        DL->getValueOrABITypeAlignment(Arg.getParamAlign(), EltTy);

    Value *Val = B.CreateExtractValue(StubCall, RetIdx++);
    B.CreateAlignedStore(Val, &Arg, Align);
  }

```
**EN:** This section contains concrete logic for BasicBlock::Create. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `PoisonValue::get`, `BasicBlock::Create`.
**CN:** 本节包含与 BasicBlock::Create 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`PoisonValue::get`, `BasicBlock::Create`。

### Lines 384-399: Declares addFnAttr
```cpp
  if (!RetTy->isVoidTy()) {
    B.CreateRet(B.CreateExtractValue(StubCall, 0));
  } else {
    B.CreateRetVoid();
  }

  // The function is now a stub we want to inline.
  F.addFnAttr(Attribute::AlwaysInline);

  ++NumOutArgumentFunctionsReplaced;
  return true;
}

FunctionPass *llvm::createAMDGPURewriteOutArgumentsPass() {
  return new AMDGPURewriteOutArguments();
}
```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `llvm::createAMDGPURewriteOutArgumentsPass`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`llvm::createAMDGPURewriteOutArgumentsPass`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPURewriteOutArguments`, `std::pair`, `cl::desc`, `cl::init`, `FunctionPass::getAnalysisUsage`, `AMDGPURewriteOutArguments::getStoredType`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPU.h"`
- `"Utils/AMDGPUBaseInfo.h"`
- `"llvm/ADT/Statistic.h"`
- `"llvm/Analysis/MemoryDependenceAnalysis.h"`
- `"llvm/IR/AttributeMask.h"`
- `"llvm/IR/IRBuilder.h"`
- `"llvm/IR/Instructions.h"`
- `"llvm/InitializePasses.h"`
- `"llvm/Pass.h"`
- `"llvm/Support/CommandLine.h"`
- `"llvm/Support/Debug.h"`
- `"llvm/Support/raw_ostream.h"`
