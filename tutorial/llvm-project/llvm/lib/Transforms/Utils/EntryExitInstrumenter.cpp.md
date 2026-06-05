# EntryExitInstrumenter.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/EntryExitInstrumenter.cpp` | `llvm/lib/Transforms/Utils/EntryExitInstrumenter.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements function Entry/Exit Instrumentation within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 EntryExitInstrumenter 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

```cpp
//===- EntryExitInstrumenter.cpp - Function Entry/Exit Instrumentation ----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/EntryExitInstrumenter.h"
#include "llvm/Analysis/GlobalsModRef.h"
#include "llvm/IR/DebugInfoMetadata.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Intrinsics.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/Type.h"
#include "llvm/InitializePasses.h"
#include "llvm/TargetParser/Triple.h"
#include "llvm/Pass.h"
#include "llvm/Transforms/Utils.h"

using namespace llvm;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。

### Lines 25-47

```cpp
static void insertCall(Function &CurFn, StringRef Func,
                       BasicBlock::iterator InsertionPt, DebugLoc DL) {
  Module &M = *InsertionPt->getParent()->getParent()->getParent();
  LLVMContext &C = InsertionPt->getParent()->getContext();

  if (Func == "mcount" ||
      Func == ".mcount" ||
      Func == "llvm.arm.gnu.eabi.mcount" ||
      Func == "\01_mcount" ||
      Func == "\01mcount" ||
      Func == "__mcount" ||
      Func == "_mcount" ||
      Func == "__cyg_profile_func_enter_bare") {
    Triple TargetTriple(M.getTargetTriple());
    if (TargetTriple.isOSAIX() && Func == "__mcount") {
      Type *SizeTy = M.getDataLayout().getIntPtrType(C);
      Type *SizePtrTy = PointerType::getUnqual(C);
      GlobalVariable *GV = new GlobalVariable(M, SizeTy, /*isConstant=*/false,
                                              GlobalValue::InternalLinkage,
                                              ConstantInt::get(SizeTy, 0));
      CallInst *Call = CallInst::Create(
          M.getOrInsertFunction(Func,
                                FunctionType::get(Type::getVoidTy(C), {SizePtrTy},
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 48-70

```cpp
                                                  /*isVarArg=*/false)),
          {GV}, "", InsertionPt);
      Call->setDebugLoc(DL);
    } else if (TargetTriple.isRISCV() || TargetTriple.isAArch64() ||
               TargetTriple.isLoongArch()) {
      // On RISC-V, AArch64, and LoongArch, the `_mcount` function takes
      // `__builtin_return_address(0)` as an argument since
      // `__builtin_return_address(1)` is not available on these platforms.
      auto ProgASPtr =
          PointerType::get(C, M.getDataLayout().getProgramAddressSpace());
      Instruction *RetAddr = CallInst::Create(
          Intrinsic::getOrInsertDeclaration(&M, Intrinsic::returnaddress,
                                            {ProgASPtr}),
          ConstantInt::get(Type::getInt32Ty(C), 0), "", InsertionPt);
      RetAddr->setDebugLoc(DL);

      FunctionCallee Fn = M.getOrInsertFunction(
          Func, FunctionType::get(Type::getVoidTy(C), PointerType::getUnqual(C),
                                  false));
      CallInst *Call = CallInst::Create(Fn, RetAddr, "", InsertionPt);
      Call->setDebugLoc(DL);
    } else if (TargetTriple.isSystemZ()) {
      // skip insertion for `mcount` on SystemZ. This will be handled later in
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 71-96

```cpp
      // `emitPrologue`. Add custom attribute to denote this.
      CurFn.addFnAttr(
          llvm::Attribute::get(C, "systemz-instrument-function-entry", Func));
    } else {
      FunctionCallee Fn = M.getOrInsertFunction(Func, Type::getVoidTy(C));
      CallInst *Call = CallInst::Create(Fn, "", InsertionPt);
      Call->setDebugLoc(DL);
    }
    return;
  }

  if (Func == "__cyg_profile_func_enter" || Func == "__cyg_profile_func_exit") {
    auto ProgASPtr =
        PointerType::get(C, M.getDataLayout().getProgramAddressSpace());
    Type *ArgTypes[] = {ProgASPtr, ProgASPtr};

    FunctionCallee Fn = M.getOrInsertFunction(
        Func, FunctionType::get(Type::getVoidTy(C), ArgTypes, false));

    Instruction *RetAddr = CallInst::Create(
        Intrinsic::getOrInsertDeclaration(&M, Intrinsic::returnaddress,
                                          {ProgASPtr}),
        ArrayRef<Value *>(ConstantInt::get(Type::getInt32Ty(C), 0)), "",
        InsertionPt);
    RetAddr->setDebugLoc(DL);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 97-122

```cpp
    Value *Args[] = {&CurFn, RetAddr};
    CallInst *Call =
        CallInst::Create(Fn, ArrayRef<Value *>(Args), "", InsertionPt);
    Call->setDebugLoc(DL);
    return;
  }

  // We only know how to call a fixed set of instrumentation functions, because
  // they all expect different arguments, etc.
  report_fatal_error(Twine("Unknown instrumentation function: '") + Func + "'");
}

static bool runOnFunction(Function &F, bool PostInlining) {
  // The asm in a naked function may reasonably expect the argument registers
  // and the return address register (if present) to be live. An inserted
  // function call will clobber these registers. Simply skip naked functions for
  // all targets.
  if (F.hasFnAttribute(Attribute::Naked))
    return false;

  // available_externally functions may not have definitions external to the
  // module (e.g. gnu::always_inline). Instrumenting them might lead to linker
  // errors if they are optimized out. Skip them like GCC.
  if (F.hasAvailableExternallyLinkage())
    return false;

```
- EN: Core entities appearing here include runOnFunction, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 runOnFunction，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 123-147

```cpp
  StringRef EntryAttr = PostInlining ? "instrument-function-entry-inlined"
                                     : "instrument-function-entry";

  StringRef ExitAttr = PostInlining ? "instrument-function-exit-inlined"
                                    : "instrument-function-exit";

  StringRef EntryFunc = F.getFnAttribute(EntryAttr).getValueAsString();
  StringRef ExitFunc = F.getFnAttribute(ExitAttr).getValueAsString();

  bool Changed = false;

  // If the attribute is specified, insert instrumentation and then "consume"
  // the attribute so that it's not inserted again if the pass should happen to
  // run later for some reason.

  if (!EntryFunc.empty()) {
    DebugLoc DL;
    if (auto SP = F.getSubprogram())
      DL = DILocation::get(SP->getContext(), SP->getScopeLine(), 0, SP);

    insertCall(F, EntryFunc, F.begin()->getFirstInsertionPt(), DL);
    Changed = true;
    F.removeFnAttr(EntryAttr);
  }

```
- EN: This region continues the EntryExitInstrumenter implementation with local helper logic centered on StringRef, EntryAttr, PostInlining, ExitAttr.
- CN: 这一段延续了 EntryExitInstrumenter 的主体实现，围绕 StringRef, EntryAttr, PostInlining, ExitAttr 等局部辅助逻辑展开。

### Lines 148-172

```cpp
  if (!ExitFunc.empty()) {
    for (BasicBlock &BB : F) {
      Instruction *T = BB.getTerminator();
      if (!isa<ReturnInst>(T))
        continue;

      // If T is preceded by a musttail call, that's the real terminator.
      if (CallInst *CI = BB.getTerminatingMustTailCall())
        T = CI;

      DebugLoc DL;
      if (DebugLoc TerminatorDL = T->getDebugLoc())
        DL = TerminatorDL;
      else if (auto SP = F.getSubprogram())
        DL = DILocation::get(SP->getContext(), 0, 0, SP);

      insertCall(F, ExitFunc, T->getIterator(), DL);
      Changed = true;
    }
    F.removeFnAttr(ExitAttr);
  }

  return Changed;
}

```
- EN: This region continues the EntryExitInstrumenter implementation with local helper logic centered on ExitFunc, BasicBlock, Instruction, ReturnInst.
- CN: 这一段延续了 EntryExitInstrumenter 的主体实现，围绕 ExitFunc, BasicBlock, Instruction, ReturnInst 等局部辅助逻辑展开。

### Lines 173-195

```cpp
namespace {
struct PostInlineEntryExitInstrumenter : public FunctionPass {
  static char ID;
  PostInlineEntryExitInstrumenter() : FunctionPass(ID) {
    initializePostInlineEntryExitInstrumenterPass(
        *PassRegistry::getPassRegistry());
  }
  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addPreserved<GlobalsAAWrapperPass>();
    AU.setPreservesCFG();
  }
  bool runOnFunction(Function &F) override { return ::runOnFunction(F, true); }
};
char PostInlineEntryExitInstrumenter::ID = 0;
}

INITIALIZE_PASS_BEGIN(
    PostInlineEntryExitInstrumenter, "post-inline-ee-instrument",
    "Instrument function entry/exit with calls to e.g. mcount() "
    "(post inlining)",
    false, false)
INITIALIZE_PASS_DEPENDENCY(DominatorTreeWrapperPass)
INITIALIZE_PASS_END(
```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include PostInlineEntryExitInstrumenter, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 PostInlineEntryExitInstrumenter，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。

### Lines 196-218

```cpp
    PostInlineEntryExitInstrumenter, "post-inline-ee-instrument",
    "Instrument function entry/exit with calls to e.g. mcount() "
    "(post inlining)",
    false, false)

FunctionPass *llvm::createPostInlineEntryExitInstrumenterPass() {
  return new PostInlineEntryExitInstrumenter();
}

PreservedAnalyses
llvm::EntryExitInstrumenterPass::run(Function &F, FunctionAnalysisManager &AM) {
  if (!runOnFunction(F, PostInlining))
    return PreservedAnalyses::all();
  PreservedAnalyses PA;
  PA.preserveSet<CFGAnalyses>();
  return PA;
}

void llvm::EntryExitInstrumenterPass::printPipeline(
    raw_ostream &OS, function_ref<StringRef(StringRef)> MapClassName2PassName) {
  static_cast<PassInfoMixin<llvm::EntryExitInstrumenterPass> *>(this)
      ->printPipeline(OS, MapClassName2PassName);
  OS << '<';
```
- EN: Core entities appearing here include run, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 run，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 219-222

```cpp
  if (PostInlining)
    OS << "post-inline";
  OS << '>';
}
```
- EN: This region continues the EntryExitInstrumenter implementation with local helper logic centered on PostInlining.
- CN: 这一段延续了 EntryExitInstrumenter 的主体实现，围绕 PostInlining 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `PostInlineEntryExitInstrumenter, runOnFunction, run` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`PostInlineEntryExitInstrumenter, runOnFunction, run` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout, DominatorTree` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout, DominatorTree` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/GlobalsModRef.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/GlobalsModRef.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/DebugInfoMetadata.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/Instructions.h`, `llvm/IR/Intrinsics.h`, `llvm/IR/Module.h`, `llvm/IR/Type.h`, `llvm/Transforms/Utils/EntryExitInstrumenter.h`, `llvm/Transforms/Utils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/DebugInfoMetadata.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/Instructions.h`, `llvm/IR/Intrinsics.h`, `llvm/IR/Module.h`, `llvm/IR/Type.h`, `llvm/Transforms/Utils/EntryExitInstrumenter.h`, `llvm/Transforms/Utils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/InitializePasses.h`, `llvm/TargetParser/Triple.h`, `llvm/Pass.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/InitializePasses.h`, `llvm/TargetParser/Triple.h`, `llvm/Pass.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `DataLayout`, `DominatorTree` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout`, `DominatorTree` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
