# AMDGPULowerKernelArguments.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPULowerKernelArguments.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPULowerKernelArguments for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPULowerKernelArguments 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: File banner, includes, and setup
```cpp
//===-- AMDGPULowerKernelArguments.cpp ------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file This pass replaces accesses to kernel arguments with loads from
/// offsets from the kernarg base pointer.
//
//===----------------------------------------------------------------------===//

#include "AMDGPU.h"
#include "AMDGPUAsanInstrumentation.h"
#include "GCNSubtarget.h"
#include "llvm/Analysis/AliasAnalysis.h"
#include "llvm/Analysis/CaptureTracking.h"
#include "llvm/Analysis/ScopedNoAliasAA.h"
#include "llvm/Analysis/ValueTracking.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/IR/Argument.h"
#include "llvm/IR/Attributes.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/InstIterator.h"
#include "llvm/IR/Instruction.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/IntrinsicsAMDGPU.h"
#include "llvm/IR/LLVMContext.h"
#include "llvm/IR/MDBuilder.h"
#include "llvm/Target/TargetMachine.h"
#include <optional>
#include <string>
```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 35-64: Preprocessor guards and macros
```cpp

#define DEBUG_TYPE "amdgpu-lower-kernel-arguments"

using namespace llvm;

namespace {

class AMDGPULowerKernelArguments : public FunctionPass {
public:
  static char ID;

  AMDGPULowerKernelArguments() : FunctionPass(ID) {}

  bool runOnFunction(Function &F) override;

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<TargetPassConfig>();
    AU.addRequired<DominatorTreeWrapperPass>();
    AU.setPreservesAll();
 }
};

} // end anonymous namespace

// skip allocas
static BasicBlock::iterator getInsertPt(BasicBlock &BB) {
  BasicBlock::iterator InsPt = BB.getFirstInsertionPt();
  for (BasicBlock::iterator E = BB.end(); InsPt != E; ++InsPt) {
    AllocaInst *AI = dyn_cast<AllocaInst>(&*InsPt);

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `AMDGPULowerKernelArguments`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`AMDGPULowerKernelArguments`。

### Lines 65-96: Defines addAliasScopeMetadata
```cpp
    // If this is a dynamic alloca, the value may depend on the loaded kernargs,
    // so loads will need to be inserted before it.
    if (!AI || !AI->isStaticAlloca())
      break;
  }

  return InsPt;
}

static void addAliasScopeMetadata(Function &F, const DataLayout &DL,
                                  DominatorTree &DT) {
  // Collect noalias arguments.
  SmallVector<const Argument *, 4u> NoAliasArgs;

  for (Argument &Arg : F.args())
    if (Arg.hasNoAliasAttr() && !Arg.use_empty())
      NoAliasArgs.push_back(&Arg);

  if (NoAliasArgs.empty())
    return;

  // Add alias scopes for each noalias argument.
  MDBuilder MDB(F.getContext());
  DenseMap<const Argument *, MDNode *> NewScopes;
  MDNode *NewDomain = MDB.createAnonymousAliasScopeDomain(F.getName());

  for (unsigned I = 0u; I < NoAliasArgs.size(); ++I) {
    const Argument *Arg = NoAliasArgs[I];
    MDNode *NewScope = MDB.createAnonymousAliasScope(NewDomain, Arg->getName());
    NewScopes.insert({Arg, NewScope});
  }

```
**EN:** This section contains concrete logic for addAliasScopeMetadata. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 addAliasScopeMetadata 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 97-125: Conditional logic and checks
```cpp
  // Iterate over all instructions.
  for (inst_iterator Inst = inst_begin(F), InstEnd = inst_end(F);
       Inst != InstEnd; ++Inst) {
    // If instruction accesses memory, collect its pointer arguments.
    Instruction *I = &(*Inst);
    SmallVector<const Value *, 2u> PtrArgs;

    if (std::optional<MemoryLocation> MO = MemoryLocation::getOrNone(I))
      PtrArgs.push_back(MO->Ptr);
    else if (const CallBase *Call = dyn_cast<CallBase>(I)) {
      if (Call->doesNotAccessMemory())
        continue;

      for (Value *Arg : Call->args()) {
        if (!Arg->getType()->isPointerTy())
          continue;

        PtrArgs.push_back(Arg);
      }
    } else {
      // Not a memory access and not a call — nothing to annotate.
      continue;
    }

    // Collect underlying objects of pointer arguments.
    SmallVector<Metadata *, 4u> Scopes;
    SmallPtrSet<const Value *, 4u> ObjSet;
    SmallVector<Metadata *, 4u> NoAliases;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `MemoryLocation::getOrNone`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`MemoryLocation::getOrNone`。

### Lines 126-157: Conditional logic and checks
```cpp
    if (!PtrArgs.empty()) {
      // Trace pointer arguments back to underlying objects and decide which
      // noalias scopes apply based on provenance and capture analysis.
      for (const Value *Val : PtrArgs) {
        SmallVector<const Value *, 4u> Objects;
        getUnderlyingObjects(Val, Objects);
        ObjSet.insert_range(Objects);
      }

      bool RequiresNoCaptureBefore = false;
      bool UsesUnknownObject = false;
      bool UsesAliasingPtr = false;

      for (const Value *Val : ObjSet) {
        if (isa<ConstantData>(Val))
          continue;

        if (const Argument *Arg = dyn_cast<Argument>(Val)) {
          if (!Arg->hasAttribute(Attribute::NoAlias))
            UsesAliasingPtr = true;
        } else
          UsesAliasingPtr = true;

        if (isEscapeSource(Val))
          RequiresNoCaptureBefore = true;
        else if (!isa<Argument>(Val) && isIdentifiedObject(Val))
          UsesUnknownObject = true;
      }

      if (UsesUnknownObject)
        continue;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 158-184: Conditional logic and checks
```cpp
      // Collect noalias scopes for instruction.
      for (const Argument *Arg : NoAliasArgs) {
        if (ObjSet.contains(Arg))
          continue;

        if (!RequiresNoCaptureBefore ||
            !capturesAnything(PointerMayBeCapturedBefore(
                Arg, false, I, &DT, false, CaptureComponents::Provenance)))
          NoAliases.push_back(NewScopes[Arg]);
      }

      // Collect scopes for alias.scope metadata.
      if (!UsesAliasingPtr)
        for (const Argument *Arg : NoAliasArgs) {
          if (ObjSet.count(Arg))
            Scopes.push_back(NewScopes[Arg]);
        }
    } else {
      // The instruction accesses memory but has no pointer arguments.
      // Since none of its operands derive from any noalias kernel argument,
      // it cannot possibly alias them. Mark it as !noalias w.r.t. every
      // noalias scope so that ScopedNoAliasAA can prove non-aliasing when
      // other instructions reference those scopes via !alias.scope.
      for (const Argument *Arg : NoAliasArgs)
        NoAliases.push_back(NewScopes[Arg]);
    }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 185-217: Conditional logic and checks
```cpp
    // Add noalias metadata to instruction.
    if (!NoAliases.empty()) {
      MDNode *NewMD =
          MDNode::concatenate(Inst->getMetadata(LLVMContext::MD_noalias),
                              MDNode::get(F.getContext(), NoAliases));
      Inst->setMetadata(LLVMContext::MD_noalias, NewMD);
    }

    // Add alias.scope metadata to instruction.
    if (!Scopes.empty()) {
      MDNode *NewMD =
          MDNode::concatenate(Inst->getMetadata(LLVMContext::MD_alias_scope),
                              MDNode::get(F.getContext(), Scopes));
      Inst->setMetadata(LLVMContext::MD_alias_scope, NewMD);
    }
  }
}

static bool lowerKernelArguments(Function &F, const TargetMachine &TM,
                                 DominatorTree &DT) {
  CallingConv::ID CC = F.getCallingConv();
  if (CC != CallingConv::AMDGPU_KERNEL || F.arg_empty())
    return false;

  const GCNSubtarget &ST = TM.getSubtarget<GCNSubtarget>(F);
  LLVMContext &Ctx = F.getContext();
  const DataLayout &DL = F.getDataLayout();
  BasicBlock &EntryBlock = *F.begin();
  IRBuilder<> Builder(&EntryBlock, getInsertPt(EntryBlock));

  const Align KernArgBaseAlign(16); // FIXME: Increase if necessary
  const uint64_t BaseOffset = ST.getExplicitKernelArgOffset();

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `MDNode::concatenate`, `MDNode::get`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`MDNode::concatenate`, `MDNode::get`。

### Lines 218-250: Declares getKernArgSegmentSize
```cpp
  Align MaxAlign;
  // FIXME: Alignment is broken with explicit arg offset.;
  const uint64_t TotalKernArgSize = ST.getKernArgSegmentSize(F, MaxAlign);
  if (TotalKernArgSize == 0)
    return false;

  CallInst *KernArgSegment =
      Builder.CreateIntrinsic(Intrinsic::amdgcn_kernarg_segment_ptr, {},
                              nullptr, F.getName() + ".kernarg.segment");
  KernArgSegment->addRetAttr(Attribute::NonNull);
  KernArgSegment->addRetAttr(
      Attribute::getWithDereferenceableBytes(Ctx, TotalKernArgSize));

  uint64_t ExplicitArgOffset = 0;

  addAliasScopeMetadata(F, F.getParent()->getDataLayout(), DT);

  for (Argument &Arg : F.args()) {
    const bool IsByRef = Arg.hasByRefAttr();
    Type *ArgTy = IsByRef ? Arg.getParamByRefType() : Arg.getType();
    MaybeAlign ParamAlign = IsByRef ? Arg.getParamAlign() : std::nullopt;
    Align ABITypeAlign = DL.getValueOrABITypeAlignment(ParamAlign, ArgTy);

    uint64_t Size = DL.getTypeSizeInBits(ArgTy);
    uint64_t AllocSize = DL.getTypeAllocSize(ArgTy);

    uint64_t EltOffset = alignTo(ExplicitArgOffset, ABITypeAlign) + BaseOffset;
    ExplicitArgOffset = alignTo(ExplicitArgOffset, ABITypeAlign) + AllocSize;

    // Skip inreg arguments which should be preloaded.
    if (Arg.use_empty() || Arg.hasInRegAttr())
      continue;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `Attribute::getWithDereferenceableBytes`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`Attribute::getWithDereferenceableBytes`。

### Lines 251-280: Conditional logic and checks
```cpp
    // If this is byval, the loads are already explicit in the function. We just
    // need to rewrite the pointer values.
    if (IsByRef) {
      Value *ArgOffsetPtr = Builder.CreateConstInBoundsGEP1_64(
          Builder.getInt8Ty(), KernArgSegment, EltOffset,
          Arg.getName() + ".byval.kernarg.offset");

      Value *CastOffsetPtr =
          Builder.CreateAddrSpaceCast(ArgOffsetPtr, Arg.getType());
      Arg.replaceAllUsesWith(CastOffsetPtr);
      continue;
    }

    if (PointerType *PT = dyn_cast<PointerType>(ArgTy)) {
      // FIXME: Hack. We rely on AssertZext to be able to fold DS addressing
      // modes on SI to know the high bits are 0 so pointer adds don't wrap. We
      // can't represent this with range metadata because it's only allowed for
      // integer types.
      if ((PT->getAddressSpace() == AMDGPUAS::LOCAL_ADDRESS ||
           PT->getAddressSpace() == AMDGPUAS::REGION_ADDRESS) &&
          !ST.hasUsableDSOffset())
        continue;
    }

    auto *VT = dyn_cast<FixedVectorType>(ArgTy);
    bool IsV3 = VT && VT->getNumElements() == 3;
    bool DoShiftOpt = Size < 32 && !ArgTy->isAggregateType();

    VectorType *V4Ty = nullptr;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 281-312: Conditional logic and checks
```cpp
    int64_t AlignDownOffset = alignDown(EltOffset, 4);
    int64_t OffsetDiff = EltOffset - AlignDownOffset;
    Align AdjustedAlign = commonAlignment(
        KernArgBaseAlign, DoShiftOpt ? AlignDownOffset : EltOffset);

    Value *ArgPtr;
    Type *AdjustedArgTy;
    if (DoShiftOpt) { // FIXME: Handle aggregate types
      // Since we don't have sub-dword scalar loads, avoid doing an extload by
      // loading earlier than the argument address, and extracting the relevant
      // bits.
      // TODO: Update this for GFX12 which does have scalar sub-dword loads.
      //
      // Additionally widen any sub-dword load to i32 even if suitably aligned,
      // so that CSE between different argument loads works easily.
      ArgPtr = Builder.CreateConstInBoundsGEP1_64(
          Builder.getInt8Ty(), KernArgSegment, AlignDownOffset,
          Arg.getName() + ".kernarg.offset.align.down");
      AdjustedArgTy = Builder.getInt32Ty();
    } else {
      ArgPtr = Builder.CreateConstInBoundsGEP1_64(
          Builder.getInt8Ty(), KernArgSegment, EltOffset,
          Arg.getName() + ".kernarg.offset");
      AdjustedArgTy = ArgTy;
    }

    if (IsV3 && Size >= 32) {
      V4Ty = FixedVectorType::get(VT->getElementType(), 4);
      // Use the hack that clang uses to avoid SelectionDAG ruining v3 loads
      AdjustedArgTy = V4Ty;
    }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `FixedVectorType::get`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`FixedVectorType::get`。

### Lines 313-341: Conditional logic and checks
```cpp
    LoadInst *Load =
        Builder.CreateAlignedLoad(AdjustedArgTy, ArgPtr, AdjustedAlign);
    Load->setMetadata(LLVMContext::MD_invariant_load, MDNode::get(Ctx, {}));

    MDBuilder MDB(Ctx);

    if (Arg.hasAttribute(Attribute::NoUndef))
      Load->setMetadata(LLVMContext::MD_noundef, MDNode::get(Ctx, {}));

    if (Arg.hasAttribute(Attribute::Range)) {
      const ConstantRange &Range =
          Arg.getAttribute(Attribute::Range).getValueAsConstantRange();
      Load->setMetadata(LLVMContext::MD_range,
                        MDB.createRange(Range.getLower(), Range.getUpper()));
    }

    if (isa<PointerType>(ArgTy)) {
      if (Arg.hasNonNullAttr())
        Load->setMetadata(LLVMContext::MD_nonnull, MDNode::get(Ctx, {}));

      uint64_t DerefBytes = Arg.getDereferenceableBytes();
      if (DerefBytes != 0) {
        Load->setMetadata(
          LLVMContext::MD_dereferenceable,
          MDNode::get(Ctx,
                      MDB.createConstant(
                        ConstantInt::get(Builder.getInt64Ty(), DerefBytes))));
      }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `MDNode::get`, `ConstantInt::get`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`MDNode::get`, `ConstantInt::get`。

### Lines 342-362: Conditional logic and checks
```cpp
      uint64_t DerefOrNullBytes = Arg.getDereferenceableOrNullBytes();
      if (DerefOrNullBytes != 0) {
        Load->setMetadata(
          LLVMContext::MD_dereferenceable_or_null,
          MDNode::get(Ctx,
                      MDB.createConstant(ConstantInt::get(Builder.getInt64Ty(),
                                                          DerefOrNullBytes))));
      }

      if (MaybeAlign ParamAlign = Arg.getParamAlign()) {
        Load->setMetadata(
            LLVMContext::MD_align,
            MDNode::get(Ctx, MDB.createConstant(ConstantInt::get(
                                 Builder.getInt64Ty(), ParamAlign->value()))));
      }
    }

    if (DoShiftOpt) {
      Value *ExtractBits = OffsetDiff == 0 ?
        Load : Builder.CreateLShr(Load, OffsetDiff * 8);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `MDNode::get`, `ConstantInt::get`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`MDNode::get`, `ConstantInt::get`。

### Lines 363-395: Registers LLVM passes
```cpp
      IntegerType *ArgIntTy = Builder.getIntNTy(Size);
      Value *Trunc = Builder.CreateTrunc(ExtractBits, ArgIntTy);
      Value *NewVal = Builder.CreateBitCast(Trunc, ArgTy,
                                            Arg.getName() + ".load");
      Arg.replaceAllUsesWith(NewVal);
    } else if (IsV3) {
      Value *Shuf = Builder.CreateShuffleVector(Load, ArrayRef<int>{0, 1, 2},
                                                Arg.getName() + ".load");
      Arg.replaceAllUsesWith(Shuf);
    } else {
      Load->setName(Arg.getName() + ".load");
      Arg.replaceAllUsesWith(Load);
    }
  }

  KernArgSegment->addRetAttr(
      Attribute::getWithAlignment(Ctx, std::max(KernArgBaseAlign, MaxAlign)));

  return true;
}

bool AMDGPULowerKernelArguments::runOnFunction(Function &F) {
  auto &TPC = getAnalysis<TargetPassConfig>();
  const TargetMachine &TM = TPC.getTM<TargetMachine>();
  DominatorTree &DT = getAnalysis<DominatorTreeWrapperPass>().getDomTree();
  return lowerKernelArguments(F, TM, DT);
}

INITIALIZE_PASS_BEGIN(AMDGPULowerKernelArguments, DEBUG_TYPE,
                      "AMDGPU Lower Kernel Arguments", false, false)
INITIALIZE_PASS_END(AMDGPULowerKernelArguments, DEBUG_TYPE, "AMDGPU Lower Kernel Arguments",
                    false, false)

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `Attribute::getWithAlignment`, `std::max`, `AMDGPULowerKernelArguments::runOnFunction`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`Attribute::getWithAlignment`, `std::max`, `AMDGPULowerKernelArguments::runOnFunction`。

### Lines 396-414: Implements llvm::createAMDGPULowerKernelArgumentsPass
```cpp
char AMDGPULowerKernelArguments::ID = 0;

FunctionPass *llvm::createAMDGPULowerKernelArgumentsPass() {
  return new AMDGPULowerKernelArguments();
}

PreservedAnalyses
AMDGPULowerKernelArgumentsPass::run(Function &F, FunctionAnalysisManager &AM) {
  DominatorTree &DT = AM.getResult<DominatorTreeAnalysis>(F);
  bool Changed = lowerKernelArguments(F, TM, DT);
  if (Changed) {
    // TODO: Preserves a lot more.
    PreservedAnalyses PA;
    PA.preserveSet<CFGAnalyses>();
    return PA;
  }

  return PreservedAnalyses::all();
}
```
**EN:** This section contains concrete logic for llvm::createAMDGPULowerKernelArgumentsPass. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `llvm::createAMDGPULowerKernelArgumentsPass`, `AMDGPULowerKernelArgumentsPass::run`, `PreservedAnalyses::all`.
**CN:** 本节包含与 llvm::createAMDGPULowerKernelArgumentsPass 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`llvm::createAMDGPULowerKernelArgumentsPass`, `AMDGPULowerKernelArgumentsPass::run`, `PreservedAnalyses::all`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPULowerKernelArguments`, `MemoryLocation::getOrNone`, `MDNode::concatenate`, `MDNode::get`, `Attribute::getWithDereferenceableBytes`, `FixedVectorType::get`
- **Main themes / 核心主题**: alias analysis / 别名分析; instruction semantics / 指令语义; subtarget modeling / 子目标建模; lowering / 降低
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPU.h"`
- `"AMDGPUAsanInstrumentation.h"`
- `"GCNSubtarget.h"`
- `"llvm/Analysis/AliasAnalysis.h"`
- `"llvm/Analysis/CaptureTracking.h"`
- `"llvm/Analysis/ScopedNoAliasAA.h"`
- `"llvm/Analysis/ValueTracking.h"`
- `"llvm/CodeGen/TargetPassConfig.h"`
- `"llvm/IR/Argument.h"`
- `"llvm/IR/Attributes.h"`
- `"llvm/IR/Dominators.h"`
- `"llvm/IR/IRBuilder.h"`
- `"llvm/IR/InstIterator.h"`
- `"llvm/IR/Instruction.h"`
- `"llvm/IR/Instructions.h"`
- `"llvm/IR/IntrinsicsAMDGPU.h"`
- `"llvm/IR/LLVMContext.h"`
- `"llvm/IR/MDBuilder.h"`
- `"llvm/Target/TargetMachine.h"`
- `<optional>`
- ... and 1 more direct dependencies / 以及另外 1 个直接依赖
