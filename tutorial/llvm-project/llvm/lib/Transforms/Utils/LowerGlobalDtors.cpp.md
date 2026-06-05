# LowerGlobalDtors.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/LowerGlobalDtors.cpp` | `llvm/lib/Transforms/Utils/LowerGlobalDtors.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements lower @llvm.global_dtors within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 LowerGlobalDtors 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

```cpp
//===-- LowerGlobalDtors.cpp - Lower @llvm.global_dtors -------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// Lower @llvm.global_dtors.
///
/// Implement @llvm.global_dtors by creating wrapper functions that are
/// registered in @llvm.global_ctors and which contain a call to
/// `__cxa_atexit` to register their destructor functions.
///
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/LowerGlobalDtors.h"

#include "llvm/IR/Constants.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Intrinsics.h"
#include "llvm/IR/Module.h"
#include "llvm/InitializePasses.h"
```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。

### Lines 25-46

```cpp
#include "llvm/Pass.h"
#include "llvm/Transforms/Utils.h"
#include "llvm/Transforms/Utils/ModuleUtils.h"
#include <map>

using namespace llvm;

#define DEBUG_TYPE "lower-global-dtors"

namespace {
class LowerGlobalDtorsLegacyPass final : public ModulePass {
  StringRef getPassName() const override {
    return "Lower @llvm.global_dtors via `__cxa_atexit`";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    ModulePass::getAnalysisUsage(AU);
  }

  bool runOnModule(Module &M) override;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 47-71

```cpp
public:
  static char ID;
  LowerGlobalDtorsLegacyPass() : ModulePass(ID) {
    initializeLowerGlobalDtorsLegacyPassPass(*PassRegistry::getPassRegistry());
  }
};
} // End anonymous namespace

char LowerGlobalDtorsLegacyPass::ID = 0;
INITIALIZE_PASS(LowerGlobalDtorsLegacyPass, DEBUG_TYPE,
                "Lower @llvm.global_dtors via `__cxa_atexit`", false, false)

ModulePass *llvm::createLowerGlobalDtorsLegacyPass() {
  return new LowerGlobalDtorsLegacyPass();
}

static bool runImpl(Module &M);
bool LowerGlobalDtorsLegacyPass::runOnModule(Module &M) { return runImpl(M); }

PreservedAnalyses LowerGlobalDtorsPass::run(Module &M,
                                            ModuleAnalysisManager &AM) {
  bool Changed = runImpl(M);
  if (!Changed)
    return PreservedAnalyses::all();

```
- EN: Core entities appearing here include LowerGlobalDtorsLegacyPass, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 LowerGlobalDtorsLegacyPass，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 72-93

```cpp
  PreservedAnalyses PA;
  PA.preserveSet<CFGAnalyses>();
  return PA;
}

static bool runImpl(Module &M) {
  GlobalVariable *GV = M.getGlobalVariable("llvm.global_dtors");
  if (!GV || !GV->hasInitializer())
    return false;

  const ConstantArray *InitList = dyn_cast<ConstantArray>(GV->getInitializer());
  if (!InitList)
    return false;

  // Validate @llvm.global_dtor's type.
  auto *ETy = dyn_cast<StructType>(InitList->getType()->getElementType());
  if (!ETy || ETy->getNumElements() != 3 ||
      !ETy->getTypeAtIndex(0U)->isIntegerTy() ||
      !ETy->getTypeAtIndex(1U)->isPointerTy() ||
      !ETy->getTypeAtIndex(2U)->isPointerTy())
    return false; // Not (int, ptr, ptr).

```
- EN: Core entities appearing here include runImpl, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 runImpl，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 94-117

```cpp
  // Collect the contents of @llvm.global_dtors, ordered by priority. Within a
  // priority, sequences of destructors with the same associated object are
  // recorded so that we can register them as a group.
  std::map<
      uint16_t,
      std::vector<std::pair<Constant *, std::vector<Constant *>>>
  > DtorFuncs;
  for (Value *O : InitList->operands()) {
    auto *CS = dyn_cast<ConstantStruct>(O);
    if (!CS)
      continue; // Malformed.

    auto *Priority = dyn_cast<ConstantInt>(CS->getOperand(0));
    if (!Priority)
      continue; // Malformed.
    uint16_t PriorityValue = Priority->getLimitedValue(UINT16_MAX);

    Constant *DtorFunc = CS->getOperand(1);
    if (DtorFunc->isNullValue())
      break; // Found a null terminator, skip the rest.

    Constant *Associated = CS->getOperand(2);
    Associated = cast<Constant>(Associated->stripPointerCasts());

```
- EN: This region continues the LowerGlobalDtors implementation with local helper logic centered on Collect, Within, Constant, DtorFuncs.
- CN: 这一段延续了 LowerGlobalDtors 的主体实现，围绕 Collect, Within, Constant, DtorFuncs 等局部辅助逻辑展开。

### Lines 118-143

```cpp
    auto &AtThisPriority = DtorFuncs[PriorityValue];
    if (AtThisPriority.empty() || AtThisPriority.back().first != Associated) {
        std::vector<Constant *> NewList;
        NewList.push_back(DtorFunc);
        AtThisPriority.push_back(std::make_pair(Associated, NewList));
    } else {
        AtThisPriority.back().second.push_back(DtorFunc);
    }
  }
  if (DtorFuncs.empty())
    return false;

  // extern "C" int __cxa_atexit(void (*f)(void *), void *p, void *d);
  LLVMContext &C = M.getContext();
  PointerType *VoidStar = PointerType::getUnqual(C);
  Type *AtExitFuncArgs[] = {VoidStar};
  FunctionType *AtExitFuncTy =
      FunctionType::get(Type::getVoidTy(C), AtExitFuncArgs,
                        /*isVarArg=*/false);

  FunctionCallee AtExit = M.getOrInsertFunction(
      "__cxa_atexit",
      FunctionType::get(Type::getInt32Ty(C),
                        {PointerType::get(C, 0), VoidStar, VoidStar},
                        /*isVarArg=*/false));

```
- EN: This region continues the LowerGlobalDtors implementation with local helper logic centered on AtThisPriority, DtorFuncs, PriorityValue, Associated.
- CN: 这一段延续了 LowerGlobalDtors 的主体实现，围绕 AtThisPriority, DtorFuncs, PriorityValue, Associated 等局部辅助逻辑展开。

### Lines 144-164

```cpp
  // If __cxa_atexit is defined (e.g. in the case of LTO) and arg0 is not
  // actually used (i.e. it's dummy/stub function as used in emscripten when
  // the program never exits) we can simply return early and clear out
  // @llvm.global_dtors.
  if (auto F = dyn_cast<Function>(AtExit.getCallee())) {
    if (F && F->hasExactDefinition() && F->getArg(0)->use_empty()) {
      GV->eraseFromParent();
      return true;
    }
  }

  // Declare __dso_local.
  Type *DsoHandleTy = Type::getInt8Ty(C);
  Constant *DsoHandle = M.getOrInsertGlobal("__dso_handle", DsoHandleTy, [&] {
    auto *GV = new GlobalVariable(M, DsoHandleTy, /*isConstant=*/true,
                                  GlobalVariable::ExternalWeakLinkage, nullptr,
                                  "__dso_handle");
    GV->setVisibility(GlobalVariable::HiddenVisibility);
    return GV;
  });

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 165-189

```cpp
  // For each unique priority level and associated symbol, generate a function
  // to call all the destructors at that level, and a function to register the
  // first function with __cxa_atexit.
  for (auto &PriorityAndMore : DtorFuncs) {
    uint16_t Priority = PriorityAndMore.first;
    uint64_t Id = 0;
    auto &AtThisPriority = PriorityAndMore.second;
    for (auto &AssociatedAndMore : AtThisPriority) {
      Constant *Associated = AssociatedAndMore.first;
      auto ThisId = Id++;

      Function *CallDtors = Function::Create(
          AtExitFuncTy, Function::PrivateLinkage,
          "call_dtors" +
              (Priority != UINT16_MAX ? (Twine(".") + Twine(Priority))
                                      : Twine()) +
              (AtThisPriority.size() > 1 ? Twine("$") + Twine(ThisId)
                                         : Twine()) +
              (!Associated->isNullValue() ? (Twine(".") + Associated->getName())
                                          : Twine()),
          &M);
      BasicBlock *BB = BasicBlock::Create(C, "body", CallDtors);
      FunctionType *VoidVoid = FunctionType::get(Type::getVoidTy(C),
                                                 /*isVarArg=*/false);

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 190-214

```cpp
      for (auto *Dtor : reverse(AssociatedAndMore.second))
        CallInst::Create(VoidVoid, Dtor, "", BB);
      ReturnInst::Create(C, BB);

      Function *RegisterCallDtors = Function::Create(
          VoidVoid, Function::PrivateLinkage,
          "register_call_dtors" +
              (Priority != UINT16_MAX ? (Twine(".") + Twine(Priority))
                                      : Twine()) +
              (AtThisPriority.size() > 1 ? Twine("$") + Twine(ThisId)
                                         : Twine()) +
              (!Associated->isNullValue() ? (Twine(".") + Associated->getName())
                                          : Twine()),
          &M);
      BasicBlock *EntryBB = BasicBlock::Create(C, "entry", RegisterCallDtors);
      BasicBlock *FailBB = BasicBlock::Create(C, "fail", RegisterCallDtors);
      BasicBlock *RetBB = BasicBlock::Create(C, "return", RegisterCallDtors);

      Value *Null = ConstantPointerNull::get(VoidStar);
      Value *Args[] = {CallDtors, Null, DsoHandle};
      Value *Res = CallInst::Create(AtExit, Args, "call", EntryBB);
      Value *Cmp = new ICmpInst(EntryBB, ICmpInst::ICMP_NE, Res,
                                Constant::getNullValue(Res->getType()));
      CondBrInst::Create(Cmp, FailBB, RetBB, EntryBB);

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 215-233

```cpp
      // If `__cxa_atexit` hits out-of-memory, trap, so that we don't misbehave.
      // This should be very rare, because if the process is running out of
      // memory before main has even started, something is wrong.
      CallInst::Create(Intrinsic::getOrInsertDeclaration(&M, Intrinsic::trap),
                       "", FailBB);
      new UnreachableInst(C, FailBB);

      ReturnInst::Create(C, RetBB);

      // Now register the registration function with @llvm.global_ctors.
      appendToGlobalCtors(M, RegisterCallDtors, Priority, Associated);
    }
  }

  // Now that we've lowered everything, remove @llvm.global_dtors.
  GV->eraseFromParent();

  return true;
}
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `LowerGlobalDtorsLegacyPass, runImpl` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`LowerGlobalDtorsLegacyPass, runImpl` 构成该文件对外 API 与主要实现挂钩。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/IR/Constants.h`, `llvm/IR/Instructions.h`, `llvm/IR/Intrinsics.h`, `llvm/IR/Module.h`, `llvm/Transforms/Utils/LowerGlobalDtors.h`, `llvm/Transforms/Utils.h`, `llvm/Transforms/Utils/ModuleUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Constants.h`, `llvm/IR/Instructions.h`, `llvm/IR/Intrinsics.h`, `llvm/IR/Module.h`, `llvm/Transforms/Utils/LowerGlobalDtors.h`, `llvm/Transforms/Utils.h`, `llvm/Transforms/Utils/ModuleUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/InitializePasses.h`, `llvm/Pass.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/InitializePasses.h`, `llvm/Pass.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `map` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`map` 为 LLVM API 之外的 C++ 语言工具提供支持。
