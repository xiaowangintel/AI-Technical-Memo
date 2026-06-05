# AMDGPUPreloadKernelArguments.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUPreloadKernelArguments.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUPreloadKernelArguments for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUPreloadKernelArguments 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33: File banner, includes, and setup
```cpp
//===- AMDGPUPreloadKernelArguments.cpp - Preload Kernel Arguments --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file This pass preloads kernel arguments into user_data SGPRs before kernel
/// execution begins. The number of registers available for preloading depends
/// on the number of free user SGPRs, up to the hardware's maximum limit.
/// Implicit arguments enabled in the kernel descriptor are allocated first,
/// followed by SGPRs used for preloaded kernel arguments. (Reference:
/// https://llvm.org/docs/AMDGPUUsage.html#initial-kernel-execution-state)
/// Additionally, hidden kernel arguments may be preloaded, in which case they
/// are appended to the kernel signature after explicit arguments. Preloaded
/// arguments will be marked with `inreg`.
//
//===----------------------------------------------------------------------===//

#include "AMDGPU.h"
#include "AMDGPUTargetMachine.h"
#include "llvm/Analysis/ValueTracking.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/IntrinsicsAMDGPU.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/PassManager.h"
#include "llvm/IR/Verifier.h"
#include "llvm/Pass.h"

#define DEBUG_TYPE "amdgpu-preload-kernel-arguments"

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 34-67: Declares class AMDGPUPreloadKernelArgumentsLegacy
```cpp
using namespace llvm;

static cl::opt<unsigned> KernargPreloadCount(
    "amdgpu-kernarg-preload-count",
    cl::desc("How many kernel arguments to preload onto SGPRs"), cl::init(0));

static cl::opt<bool>
    EnableKernargPreload("amdgpu-kernarg-preload",
                         cl::desc("Enable preload kernel arguments to SGPRs"),
                         cl::init(true));

namespace {

class AMDGPUPreloadKernelArgumentsLegacy : public ModulePass {
  const GCNTargetMachine *TM;

public:
  static char ID;
  explicit AMDGPUPreloadKernelArgumentsLegacy(
      const GCNTargetMachine *TM = nullptr);

  StringRef getPassName() const override {
    return "AMDGPU Preload Kernel Arguments";
  }

  bool runOnModule(Module &M) override;
};

class PreloadKernelArgInfo {
private:
  Function &F;
  const GCNSubtarget &ST;
  unsigned NumFreeUserSGPRs;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUPreloadKernelArgumentsLegacy`, `PreloadKernelArgInfo`, `cl::desc`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUPreloadKernelArgumentsLegacy`, `PreloadKernelArgInfo`, `cl::desc`。

### Lines 68-98: Declares struct HiddenArgInfo
```cpp
  enum HiddenArg : unsigned {
    HIDDEN_BLOCK_COUNT_X,
    HIDDEN_BLOCK_COUNT_Y,
    HIDDEN_BLOCK_COUNT_Z,
    HIDDEN_GROUP_SIZE_X,
    HIDDEN_GROUP_SIZE_Y,
    HIDDEN_GROUP_SIZE_Z,
    HIDDEN_REMAINDER_X,
    HIDDEN_REMAINDER_Y,
    HIDDEN_REMAINDER_Z,
    END_HIDDEN_ARGS
  };

  // Stores information about a specific hidden argument.
  struct HiddenArgInfo {
    // Offset in bytes from the location in the kernearg segment pointed to by
    // the implicitarg pointer.
    uint8_t Offset;
    // The size of the hidden argument in bytes.
    uint8_t Size;
    // The name of the hidden argument in the kernel signature.
    const char *Name;
  };

  static constexpr HiddenArgInfo HiddenArgs[END_HIDDEN_ARGS] = {
      {0, 4, "_hidden_block_count_x"}, {4, 4, "_hidden_block_count_y"},
      {8, 4, "_hidden_block_count_z"}, {12, 2, "_hidden_group_size_x"},
      {14, 2, "_hidden_group_size_y"}, {16, 2, "_hidden_group_size_z"},
      {18, 2, "_hidden_remainder_x"},  {20, 2, "_hidden_remainder_y"},
      {22, 2, "_hidden_remainder_z"}};

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `HiddenArg`, `HiddenArgInfo`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`HiddenArg`, `HiddenArgInfo`。

### Lines 99-120: Defines getHiddenArgFromOffset
```cpp
  static HiddenArg getHiddenArgFromOffset(unsigned Offset) {
    for (unsigned I = 0; I < END_HIDDEN_ARGS; ++I)
      if (HiddenArgs[I].Offset == Offset)
        return static_cast<HiddenArg>(I);

    return END_HIDDEN_ARGS;
  }

  static Type *getHiddenArgType(LLVMContext &Ctx, HiddenArg HA) {
    if (HA < END_HIDDEN_ARGS)
      return Type::getIntNTy(Ctx, HiddenArgs[HA].Size * 8);

    llvm_unreachable("Unexpected hidden argument.");
  }

  static const char *getHiddenArgName(HiddenArg HA) {
    if (HA < END_HIDDEN_ARGS)
      return HiddenArgs[HA].Name;

    llvm_unreachable("Unexpected hidden argument.");
  }

```
**EN:** This section contains concrete logic for getHiddenArgFromOffset. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `Type::getIntNTy`.
**CN:** 本节包含与 getHiddenArgFromOffset 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`Type::getIntNTy`。

### Lines 121-153: Defines cloneFunctionWithPreloadImplicitArgs
```cpp
  // Clones the function after adding implicit arguments to the argument list
  // and returns the new updated function. Preloaded implicit arguments are
  // added up to and including the last one that will be preloaded, indicated by
  // LastPreloadIndex. Currently preloading is only performed on the totality of
  // sequential data from the kernarg segment including implicit (hidden)
  // arguments. This means that all arguments up to the last preloaded argument
  // will also be preloaded even if that data is unused.
  Function *cloneFunctionWithPreloadImplicitArgs(unsigned LastPreloadIndex) {
    FunctionType *FT = F.getFunctionType();
    LLVMContext &Ctx = F.getContext();
    SmallVector<Type *, 16> FTypes(FT->param_begin(), FT->param_end());
    for (unsigned I = 0; I <= LastPreloadIndex; ++I)
      FTypes.push_back(getHiddenArgType(Ctx, HiddenArg(I)));

    FunctionType *NFT =
        FunctionType::get(FT->getReturnType(), FTypes, FT->isVarArg());
    Function *NF =
        Function::Create(NFT, F.getLinkage(), F.getAddressSpace(), F.getName());

    NF->copyAttributesFrom(&F);
    NF->copyMetadata(&F, 0);

    F.getParent()->getFunctionList().insert(F.getIterator(), NF);
    NF->takeName(&F);
    NF->splice(NF->begin(), &F);

    Function::arg_iterator NFArg = NF->arg_begin();
    for (Argument &Arg : F.args()) {
      Arg.replaceAllUsesWith(&*NFArg);
      NFArg->takeName(&Arg);
      ++NFArg;
    }

```
**EN:** This section contains concrete logic for cloneFunctionWithPreloadImplicitArgs. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `FunctionType::get`, `Function::Create`.
**CN:** 本节包含与 cloneFunctionWithPreloadImplicitArgs 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`FunctionType::get`, `Function::Create`。

### Lines 154-184: Result computation and returns
```cpp
    AttrBuilder AB(Ctx);
    AB.addAttribute(Attribute::InReg);
    AB.addAttribute("amdgpu-hidden-argument");
    AttributeList AL = NF->getAttributes();
    for (unsigned I = 0; I <= LastPreloadIndex; ++I) {
      AL = AL.addParamAttributes(Ctx, NFArg->getArgNo(), AB);
      NFArg++->setName(getHiddenArgName(HiddenArg(I)));
    }

    NF->setAttributes(AL);
    F.replaceAllUsesWith(NF);

    return NF;
  }

public:
  PreloadKernelArgInfo(Function &F, const GCNSubtarget &ST) : F(F), ST(ST) {
    setInitialFreeUserSGPRsCount();
  }

  // Returns the maximum number of user SGPRs that we have available to preload
  // arguments.
  void setInitialFreeUserSGPRsCount() {
    GCNUserSGPRUsageInfo UserSGPRInfo(F, ST);
    NumFreeUserSGPRs = UserSGPRInfo.getNumFreeUserSGPRs();
  }

  bool canPreloadKernArgAtOffset(uint64_t ExplicitArgOffset) {
    return ExplicitArgOffset <= NumFreeUserSGPRs * 4;
  }

```
**EN:** This section finalizes intermediate state and returns the value that the caller or pass pipeline needs.
**CN:** 本节整理中间状态并返回调用者或 Pass 流水线所需的结果。

### Lines 185-214: Defines tryAllocHiddenArgPreloadSGPRs
```cpp
  // Try to allocate SGPRs to preload hidden kernel arguments.
  void
  tryAllocHiddenArgPreloadSGPRs(uint64_t ImplicitArgsBaseOffset,
                                SmallVectorImpl<Function *> &FunctionsToErase) {
    Function *ImplicitArgPtr = Intrinsic::getDeclarationIfExists(
        F.getParent(), Intrinsic::amdgcn_implicitarg_ptr);
    if (!ImplicitArgPtr)
      return;

    const DataLayout &DL = F.getParent()->getDataLayout();
    // Pair is the load and the load offset.
    SmallVector<std::pair<LoadInst *, unsigned>, 4> ImplicitArgLoads;
    for (auto *U : ImplicitArgPtr->users()) {
      Instruction *CI = dyn_cast<Instruction>(U);
      if (!CI || CI->getFunction() != &F)
        continue;

      for (auto *U : CI->users()) {
        int64_t Offset = 0;
        auto *Load = dyn_cast<LoadInst>(U); // Load from ImplicitArgPtr?
        if (!Load) {
          if (GetPointerBaseWithConstantOffset(U, Offset, DL) != CI)
            continue;

          Load = dyn_cast<LoadInst>(*U->user_begin()); // Load from GEP?
        }

        if (!Load || !Load->isSimple())
          continue;

```
**EN:** This section contains concrete logic for tryAllocHiddenArgPreloadSGPRs. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `Intrinsic::getDeclarationIfExists`.
**CN:** 本节包含与 tryAllocHiddenArgPreloadSGPRs 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`Intrinsic::getDeclarationIfExists`。

### Lines 215-246: Conditional logic and checks
```cpp
        // FIXME: Expand handle merged loads.
        LLVMContext &Ctx = F.getContext();
        Type *LoadTy = Load->getType();
        HiddenArg HA = getHiddenArgFromOffset(Offset);
        if (HA == END_HIDDEN_ARGS || LoadTy != getHiddenArgType(Ctx, HA))
          continue;

        ImplicitArgLoads.push_back(std::make_pair(Load, Offset));
      }
    }

    if (ImplicitArgLoads.empty())
      return;

    // Allocate loads in order of offset. We need to be sure that the implicit
    // argument can actually be preloaded.
    std::sort(ImplicitArgLoads.begin(), ImplicitArgLoads.end(), less_second());

    // If we fail to preload any implicit argument we know we don't have SGPRs
    // to preload any subsequent ones with larger offsets. Find the first
    // argument that we cannot preload.
    auto *PreloadEnd = llvm::find_if(
        ImplicitArgLoads, [&](const std::pair<LoadInst *, unsigned> &Load) {
          unsigned LoadSize = DL.getTypeStoreSize(Load.first->getType());
          unsigned LoadOffset = Load.second;
          if (!canPreloadKernArgAtOffset(LoadOffset + LoadSize +
                                         ImplicitArgsBaseOffset))
            return true;

          return false;
        });

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::make_pair`, `std::sort`, `llvm::find_if`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::make_pair`, `std::sort`, `llvm::find_if`。

### Lines 247-277: Registers LLVM passes
```cpp
    if (PreloadEnd == ImplicitArgLoads.begin())
      return;

    unsigned LastHiddenArgIndex = getHiddenArgFromOffset(PreloadEnd[-1].second);
    Function *NF = cloneFunctionWithPreloadImplicitArgs(LastHiddenArgIndex);
    assert(NF);
    FunctionsToErase.push_back(&F);
    for (const auto *I = ImplicitArgLoads.begin(); I != PreloadEnd; ++I) {
      LoadInst *LoadInst = I->first;
      unsigned LoadOffset = I->second;
      unsigned HiddenArgIndex = getHiddenArgFromOffset(LoadOffset);
      unsigned Index = NF->arg_size() - LastHiddenArgIndex + HiddenArgIndex - 1;
      Argument *Arg = NF->getArg(Index);
      LoadInst->replaceAllUsesWith(Arg);
    }
  }
};

} // end anonymous namespace

char AMDGPUPreloadKernelArgumentsLegacy::ID = 0;

INITIALIZE_PASS(AMDGPUPreloadKernelArgumentsLegacy, DEBUG_TYPE,
                "AMDGPU Preload Kernel Arguments", false, false)

ModulePass *
llvm::createAMDGPUPreloadKernelArgumentsLegacyPass(const TargetMachine *TM) {
  return new AMDGPUPreloadKernelArgumentsLegacy(
      static_cast<const GCNTargetMachine *>(TM));
}

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `llvm::createAMDGPUPreloadKernelArgumentsLegacyPass`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`llvm::createAMDGPUPreloadKernelArgumentsLegacyPass`。

### Lines 278-308: Implements AMDGPUPreloadKernelArgumentsLegacy::AMDGPUPreloadKernelArgumentsLegacy
```cpp
AMDGPUPreloadKernelArgumentsLegacy::AMDGPUPreloadKernelArgumentsLegacy(
    const GCNTargetMachine *TM)
    : ModulePass(ID), TM(TM) {}

static bool markKernelArgsAsInreg(Module &M, const TargetMachine &TM) {
  if (!EnableKernargPreload)
    return false;

  SmallVector<Function *, 4> FunctionsToErase;
  bool Changed = false;
  for (auto &F : M) {
    const GCNSubtarget &ST = TM.getSubtarget<GCNSubtarget>(F);
    if (!ST.hasKernargPreload() ||
        F.getCallingConv() != CallingConv::AMDGPU_KERNEL)
      continue;

    PreloadKernelArgInfo PreloadInfo(F, ST);
    uint64_t ExplicitArgOffset = 0;
    const DataLayout &DL = F.getDataLayout();
    const uint64_t BaseOffset = ST.getExplicitKernelArgOffset();
    unsigned NumPreloadsRequested = KernargPreloadCount;
    unsigned NumPreloadedExplicitArgs = 0;
    for (Argument &Arg : F.args()) {
      // Avoid incompatible attributes and guard against running this pass
      // twice.
      //
      // TODO: Preload byref kernel arguments
      if (Arg.hasByRefAttr() || Arg.hasNestAttr() ||
          Arg.hasAttribute("amdgpu-hidden-argument"))
        break;

```
**EN:** This section contains concrete logic for AMDGPUPreloadKernelArgumentsLegacy::AMDGPUPreloadKernelArgumentsLegacy. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUPreloadKernelArgumentsLegacy::AMDGPUPreloadKernelArgumentsLegacy`.
**CN:** 本节包含与 AMDGPUPreloadKernelArgumentsLegacy::AMDGPUPreloadKernelArgumentsLegacy 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUPreloadKernelArgumentsLegacy::AMDGPUPreloadKernelArgumentsLegacy`。

### Lines 309-340: Conditional logic and checks
```cpp
      // Inreg may be pre-existing on some arguments, try to preload these.
      if (NumPreloadsRequested == 0 && !Arg.hasInRegAttr())
        break;

      // FIXME: Preload aggregates.
      if (Arg.getType()->isAggregateType())
        break;

      Type *ArgTy = Arg.getType();
      Align ABITypeAlign = DL.getABITypeAlign(ArgTy);
      uint64_t AllocSize = DL.getTypeAllocSize(ArgTy);
      ExplicitArgOffset = alignTo(ExplicitArgOffset, ABITypeAlign) + AllocSize;

      if (!PreloadInfo.canPreloadKernArgAtOffset(ExplicitArgOffset))
        break;

      Arg.addAttr(Attribute::InReg);
      NumPreloadedExplicitArgs++;
      if (NumPreloadsRequested > 0)
        NumPreloadsRequested--;
    }

    // Only try preloading hidden arguments if we can successfully preload the
    // last explicit argument.
    if (NumPreloadedExplicitArgs == F.arg_size()) {
      uint64_t ImplicitArgsBaseOffset =
          alignTo(ExplicitArgOffset, ST.getAlignmentForImplicitArgPtr()) +
          BaseOffset;
      PreloadInfo.tryAllocHiddenArgPreloadSGPRs(ImplicitArgsBaseOffset,
                                                FunctionsToErase);
    }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 341-364: Declares empty
```cpp
    Changed |= NumPreloadedExplicitArgs > 0;
  }

  Changed |= !FunctionsToErase.empty();
  // Erase cloned functions if we needed to update the kernel signature to
  // support preloading hidden kernel arguments.
  for (auto *F : FunctionsToErase)
    F->eraseFromParent();

  return Changed;
}

bool AMDGPUPreloadKernelArgumentsLegacy::runOnModule(Module &M) {
  if (skipModule(M) || !TM)
    return false;

  return markKernelArgsAsInreg(M, *TM);
}

PreservedAnalyses
AMDGPUPreloadKernelArgumentsPass::run(Module &M, ModuleAnalysisManager &AM) {
  bool Changed = markKernelArgsAsInreg(M, TM);
  return Changed ? PreservedAnalyses::none() : PreservedAnalyses::all();
}
```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `AMDGPUPreloadKernelArgumentsLegacy::runOnModule`, `AMDGPUPreloadKernelArgumentsPass::run`, `PreservedAnalyses::none`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`AMDGPUPreloadKernelArgumentsLegacy::runOnModule`, `AMDGPUPreloadKernelArgumentsPass::run`, `PreservedAnalyses::none`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPUPreloadKernelArgumentsLegacy`, `PreloadKernelArgInfo`, `HiddenArg`, `HiddenArgInfo`, `cl::desc`, `cl::init`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPU.h"`
- `"AMDGPUTargetMachine.h"`
- `"llvm/Analysis/ValueTracking.h"`
- `"llvm/IR/Function.h"`
- `"llvm/IR/Instructions.h"`
- `"llvm/IR/IntrinsicsAMDGPU.h"`
- `"llvm/IR/Module.h"`
- `"llvm/IR/PassManager.h"`
- `"llvm/IR/Verifier.h"`
- `"llvm/Pass.h"`
