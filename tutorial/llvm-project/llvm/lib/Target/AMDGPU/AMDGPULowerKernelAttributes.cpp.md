# AMDGPULowerKernelAttributes.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPULowerKernelAttributes.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPULowerKernelAttributes for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPULowerKernelAttributes 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33: File banner, includes, and setup
```cpp
//===-- AMDGPULowerKernelAttributes.cpp------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file This pass does attempts to make use of reqd_work_group_size metadata
/// to eliminate loads from the dispatch packet and to constant fold OpenCL
/// get_local_size-like functions.
//
//===----------------------------------------------------------------------===//

#include "AMDGPU.h"
#include "Utils/AMDGPUBaseInfo.h"
#include "llvm/Analysis/ConstantFolding.h"
#include "llvm/Analysis/ValueTracking.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/InstIterator.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/IntrinsicsAMDGPU.h"
#include "llvm/IR/MDBuilder.h"
#include "llvm/IR/PatternMatch.h"
#include "llvm/Pass.h"

#define DEBUG_TYPE "amdgpu-lower-kernel-attributes"

using namespace llvm;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 34-67: Declares class AMDGPULowerKernelAttributes
```cpp
namespace {

// Field offsets in hsa_kernel_dispatch_packet_t.
enum DispatchPackedOffsets {
  WORKGROUP_SIZE_X = 4,
  WORKGROUP_SIZE_Y = 6,
  WORKGROUP_SIZE_Z = 8,

  GRID_SIZE_X = 12,
  GRID_SIZE_Y = 16,
  GRID_SIZE_Z = 20
};

// Field offsets to implicit kernel argument pointer.
enum ImplicitArgOffsets {
  HIDDEN_BLOCK_COUNT_X = 0,
  HIDDEN_BLOCK_COUNT_Y = 4,
  HIDDEN_BLOCK_COUNT_Z = 8,

  HIDDEN_GROUP_SIZE_X = 12,
  HIDDEN_GROUP_SIZE_Y = 14,
  HIDDEN_GROUP_SIZE_Z = 16,

  HIDDEN_REMAINDER_X = 18,
  HIDDEN_REMAINDER_Y = 20,
  HIDDEN_REMAINDER_Z = 22,

  GRID_DIMS = 64
};

class AMDGPULowerKernelAttributes : public ModulePass {
public:
  static char ID;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `DispatchPackedOffsets`, `ImplicitArgOffsets`, `AMDGPULowerKernelAttributes`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`DispatchPackedOffsets`, `ImplicitArgOffsets`, `AMDGPULowerKernelAttributes`。

### Lines 68-98: Defines AMDGPULowerKernelAttributes
```cpp
  AMDGPULowerKernelAttributes() : ModulePass(ID) {}

  bool runOnModule(Module &M) override;

  StringRef getPassName() const override { return "AMDGPU Kernel Attributes"; }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesAll();
  }
};

Function *getBasePtrIntrinsic(Module &M, bool IsV5OrAbove) {
  auto IntrinsicId = IsV5OrAbove ? Intrinsic::amdgcn_implicitarg_ptr
                                 : Intrinsic::amdgcn_dispatch_ptr;
  return Intrinsic::getDeclarationIfExists(&M, IntrinsicId);
}

} // end anonymous namespace

static bool annotateGridSizeLoadWithRangeMD(LoadInst *Load,
                                            uint32_t MaxNumGroups) {
  if (MaxNumGroups == 0 || MaxNumGroups == std::numeric_limits<uint32_t>::max())
    return false;

  if (!Load->getType()->isIntegerTy(32))
    return false;

  // TODO: If there is existing range metadata, preserve it if it is stricter.
  if (Load->hasMetadata(LLVMContext::MD_range))
    return false;

```
**EN:** This section contains concrete logic for AMDGPULowerKernelAttributes. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `Intrinsic::getDeclarationIfExists`.
**CN:** 本节包含与 AMDGPULowerKernelAttributes 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`Intrinsic::getDeclarationIfExists`。

### Lines 99-132: Declares MDB
```cpp
  MDBuilder MDB(Load->getContext());
  MDNode *Range = MDB.createRange(APInt(32, 1), APInt(32, MaxNumGroups + 1));
  Load->setMetadata(LLVMContext::MD_range, Range);
  return true;
}

static bool annotateGroupSizeLoadWithRangeMD(LoadInst *Load, bool IsRemainder) {
  if (!Load->getType()->isIntegerTy(16))
    return false;

  // TODO: If there is existing range metadata, preserve it if it is stricter.
  if (Load->hasMetadata(LLVMContext::MD_range))
    return false;

  MDBuilder MDB(Load->getContext());
  MDNode *Range = MDB.createRange(
      APInt(16, !IsRemainder),
      APInt(16, AMDGPU::IsaInfo::getMaxFlatWorkGroupSize() + 1 - IsRemainder));
  Load->setMetadata(LLVMContext::MD_range, Range);
  return true;
}

static bool annotateGridDimsLoadWithRangeMD(LoadInst *Load,
                                            unsigned KnownNumGridDims) {
  IntegerType *Ty = dyn_cast<IntegerType>(Load->getType());
  if (!Ty || Ty->getBitWidth() < 3)
    return false;

  if (KnownNumGridDims != 0) {
    Load->replaceAllUsesWith(
        ConstantInt::get(Load->getType(), KnownNumGridDims));
    return true;
  }

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `IsaInfo::getMaxFlatWorkGroupSize`, `ConstantInt::get`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`IsaInfo::getMaxFlatWorkGroupSize`, `ConstantInt::get`。

### Lines 133-165: Declares MDB
```cpp
  // TODO: If there is existing range metadata, preserve it if it is stricter.
  if (Load->hasMetadata(LLVMContext::MD_range))
    return false;

  MDBuilder MDB(Load->getContext());
  MDNode *Range =
      MDB.createRange(APInt(Ty->getBitWidth(), 1), APInt(Ty->getBitWidth(), 4));
  Load->setMetadata(LLVMContext::MD_range, Range);
  return true;
}

/// Compute the number of grid dimensions based on !reqd_work_group_size
/// metadata
static unsigned computeNumGridDims(const MDNode *ReqdWorkGroupSize) {
  ConstantInt *KnownZ =
      mdconst::extract<ConstantInt>(ReqdWorkGroupSize->getOperand(2));
  if (KnownZ->getZExtValue() != 1)
    return 3;

  ConstantInt *KnownY =
      mdconst::extract<ConstantInt>(ReqdWorkGroupSize->getOperand(1));
  if (KnownY->getZExtValue() != 1)
    return 2;

  return 1;
}

static bool processUse(CallInst *CI, bool IsV5OrAbove) {
  Function *F = CI->getFunction();

  auto *MD = F->getMetadata("reqd_work_group_size");
  const bool HasReqdWorkGroupSize = MD && MD->getNumOperands() == 3;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 166-198: Conditional logic and checks
```cpp
  const bool HasUniformWorkGroupSize =
      F->hasFnAttribute("uniform-work-group-size");

  SmallVector<unsigned> MaxNumWorkgroups =
      AMDGPU::getIntegerVecAttribute(*F, "amdgpu-max-num-workgroups",
                                     /*Size=*/3, /*DefaultVal=*/0);

  Value *BlockCounts[3] = {nullptr, nullptr, nullptr};
  Value *GroupSizes[3] = {nullptr, nullptr, nullptr};
  Value *Remainders[3] = {nullptr, nullptr, nullptr};
  Value *GridSizes[3] = {nullptr, nullptr, nullptr};

  const DataLayout &DL = F->getDataLayout();
  bool MadeChange = false;

  unsigned KnownNumGridDims = HasReqdWorkGroupSize ? computeNumGridDims(MD) : 0;

  // We expect to see several GEP users, casted to the appropriate type and
  // loaded.
  for (User *U : CI->users()) {
    if (!U->hasOneUse())
      continue;

    int64_t Offset = 0;
    auto *Load = dyn_cast<LoadInst>(U); // Load from ImplicitArgPtr/DispatchPtr?
    auto *BCI = dyn_cast<BitCastInst>(U);
    if (!Load && !BCI) {
      if (GetPointerBaseWithConstantOffset(U, Offset, DL) != CI)
        continue;
      Load = dyn_cast<LoadInst>(*U->user_begin()); // Load from GEP?
      BCI = dyn_cast<BitCastInst>(*U->user_begin());
    }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::getIntegerVecAttribute`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::getIntegerVecAttribute`。

### Lines 199-232: Switch-based control flow
```cpp
    if (BCI) {
      if (!BCI->hasOneUse())
        continue;
      Load = dyn_cast<LoadInst>(*BCI->user_begin()); // Load from BCI?
    }

    if (!Load || !Load->isSimple())
      continue;

    unsigned LoadSize = DL.getTypeStoreSize(Load->getType());

    // TODO: Handle merged loads.
    if (IsV5OrAbove) { // Base is ImplicitArgPtr.
      switch (Offset) {
      case HIDDEN_BLOCK_COUNT_X:
        if (LoadSize == 4) {
          BlockCounts[0] = Load;
          MadeChange |=
              annotateGridSizeLoadWithRangeMD(Load, MaxNumWorkgroups[0]);
        }
        break;
      case HIDDEN_BLOCK_COUNT_Y:
        if (LoadSize == 4) {
          BlockCounts[1] = Load;
          MadeChange |=
              annotateGridSizeLoadWithRangeMD(Load, MaxNumWorkgroups[1]);
        }
        break;
      case HIDDEN_BLOCK_COUNT_Z:
        if (LoadSize == 4) {
          BlockCounts[2] = Load;
          MadeChange |=
              annotateGridSizeLoadWithRangeMD(Load, MaxNumWorkgroups[2]);
        }
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 233-266: Conditional logic and checks
```cpp
        break;
      case HIDDEN_GROUP_SIZE_X:
        if (LoadSize == 2) {
          GroupSizes[0] = Load;
          MadeChange |= annotateGroupSizeLoadWithRangeMD(Load, false);
        }
        break;
      case HIDDEN_GROUP_SIZE_Y:
        if (LoadSize == 2) {
          GroupSizes[1] = Load;
          MadeChange |= annotateGroupSizeLoadWithRangeMD(Load, false);
        }
        break;
      case HIDDEN_GROUP_SIZE_Z:
        if (LoadSize == 2) {
          GroupSizes[2] = Load;
          MadeChange |= annotateGroupSizeLoadWithRangeMD(Load, false);
        }
        break;
      case HIDDEN_REMAINDER_X:
        if (LoadSize == 2) {
          Remainders[0] = Load;
          MadeChange |= annotateGroupSizeLoadWithRangeMD(Load, true);
        }
        break;
      case HIDDEN_REMAINDER_Y:
        if (LoadSize == 2) {
          Remainders[1] = Load;
          MadeChange |= annotateGroupSizeLoadWithRangeMD(Load, true);
        }
        break;
      case HIDDEN_REMAINDER_Z:
        if (LoadSize == 2) {
          Remainders[2] = Load;
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 267-300: Switch-based control flow
```cpp
          MadeChange |= annotateGroupSizeLoadWithRangeMD(Load, true);
        }
        break;

      case GRID_DIMS:
        if (LoadSize <= 2)
          MadeChange |= annotateGridDimsLoadWithRangeMD(Load, KnownNumGridDims);
        break;
      default:
        break;
      }
    } else { // Base is DispatchPtr.
      switch (Offset) {
      case WORKGROUP_SIZE_X:
        if (LoadSize == 2)
          GroupSizes[0] = Load;
        break;
      case WORKGROUP_SIZE_Y:
        if (LoadSize == 2)
          GroupSizes[1] = Load;
        break;
      case WORKGROUP_SIZE_Z:
        if (LoadSize == 2)
          GroupSizes[2] = Load;
        break;
      case GRID_SIZE_X:
        if (LoadSize == 4)
          GridSizes[0] = Load;
        break;
      case GRID_SIZE_Y:
        if (LoadSize == 4)
          GridSizes[1] = Load;
        break;
      case GRID_SIZE_Z:
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 301-330: Namespace declarations and scope setup
```cpp
        if (LoadSize == 4)
          GridSizes[2] = Load;
        break;
      default:
        break;
      }
    }
  }

  if (IsV5OrAbove && HasUniformWorkGroupSize) {
    // Under v5  __ockl_get_local_size returns the value computed by the
    // expression:
    //
    //   workgroup_id < hidden_block_count ? hidden_group_size :
    //                                       hidden_remainder
    //
    // For functions with the attribute uniform-work-group-size=true. we can
    // evaluate workgroup_id < hidden_block_count as true, and thus
    // hidden_group_size is returned for __ockl_get_local_size.
    for (int I = 0; I < 3; ++I) {
      Value *BlockCount = BlockCounts[I];
      if (!BlockCount)
        continue;

      using namespace llvm::PatternMatch;
      auto GroupIDIntrin =
          I == 0 ? m_Intrinsic<Intrinsic::amdgcn_workgroup_id_x>()
                 : (I == 1 ? m_Intrinsic<Intrinsic::amdgcn_workgroup_id_y>()
                           : m_Intrinsic<Intrinsic::amdgcn_workgroup_id_z>());

```
**EN:** This section arranges declarations into namespace scope so related AMDGPU functionality stays organized and avoids symbol collisions.
**CN:** 本节将声明组织到命名空间作用域中，使相关 AMDGPU 功能保持清晰并避免符号冲突。

### Lines 331-364: Conditional logic and checks
```cpp
      for (User *ICmp : BlockCount->users()) {
        if (match(ICmp, m_SpecificICmp(ICmpInst::ICMP_ULT, GroupIDIntrin,
                                       m_Specific(BlockCount)))) {
          ICmp->replaceAllUsesWith(llvm::ConstantInt::getTrue(ICmp->getType()));
          MadeChange = true;
        }
      }
    }

    // All remainders should be 0 with uniform work group size.
    for (Value *Remainder : Remainders) {
      if (!Remainder)
        continue;
      Remainder->replaceAllUsesWith(
          Constant::getNullValue(Remainder->getType()));
      MadeChange = true;
    }
  } else if (HasUniformWorkGroupSize) { // Pre-V5.
    // Pattern match the code used to handle partial workgroup dispatches in the
    // library implementation of get_local_size, so the entire function can be
    // constant folded with a known group size.
    //
    // uint r = grid_size - group_id * group_size;
    // get_local_size = (r < group_size) ? r : group_size;
    //
    // If we have uniform-work-group-size (which is the default in OpenCL 1.2),
    // the grid_size is required to be a multiple of group_size). In this case:
    //
    // grid_size - (group_id * group_size) < group_size
    // ->
    // grid_size < group_size + (group_id * group_size)
    //
    // (grid_size / group_size) < 1 + group_id
    //
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `ConstantInt::getTrue`, `Constant::getNullValue`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`ConstantInt::getTrue`, `Constant::getNullValue`。

### Lines 365-398: Namespace declarations and scope setup
```cpp
    // grid_size / group_size is at least 1, so we can conclude the select
    // condition is false (except for group_id == 0, where the select result is
    // the same).
    for (int I = 0; I < 3; ++I) {
      Value *GroupSize = GroupSizes[I];
      Value *GridSize = GridSizes[I];
      if (!GroupSize || !GridSize)
        continue;

      using namespace llvm::PatternMatch;
      auto GroupIDIntrin =
          I == 0 ? m_Intrinsic<Intrinsic::amdgcn_workgroup_id_x>()
                 : (I == 1 ? m_Intrinsic<Intrinsic::amdgcn_workgroup_id_y>()
                           : m_Intrinsic<Intrinsic::amdgcn_workgroup_id_z>());

      for (User *U : GroupSize->users()) {
        auto *ZextGroupSize = dyn_cast<ZExtInst>(U);
        if (!ZextGroupSize)
          continue;

        for (User *UMin : ZextGroupSize->users()) {
          if (match(UMin, m_UMin(m_Sub(m_Specific(GridSize),
                                       m_Mul(GroupIDIntrin,
                                             m_Specific(ZextGroupSize))),
                                 m_Specific(ZextGroupSize)))) {
            if (HasReqdWorkGroupSize) {
              ConstantInt *KnownSize =
                  mdconst::extract<ConstantInt>(MD->getOperand(I));
              UMin->replaceAllUsesWith(ConstantFoldIntegerCast(
                  KnownSize, UMin->getType(), false, DL));
            } else {
              UMin->replaceAllUsesWith(ZextGroupSize);
            }

```
**EN:** This section arranges declarations into namespace scope so related AMDGPU functionality stays organized and avoids symbol collisions.
**CN:** 本节将声明组织到命名空间作用域中，使相关 AMDGPU 功能保持清晰并避免符号冲突。

### Lines 399-430: Namespace declarations and scope setup
```cpp
            MadeChange = true;
          }
        }
      }
    }
  }

  // Upgrade the old method of calculating the block size using the grid size.
  // We pattern match any case where the implicit argument group size is the
  // divisor to a dispatch packet grid size read of the same dimension.
  if (IsV5OrAbove) {
    for (int I = 0; I < 3; I++) {
      Value *GroupSize = GroupSizes[I];
      if (!GroupSize || !GroupSize->getType()->isIntegerTy(16))
        continue;

      for (User *U : GroupSize->users()) {
        Instruction *Inst = cast<Instruction>(U);
        if (isa<ZExtInst>(Inst) && !Inst->use_empty())
          Inst = cast<Instruction>(*Inst->user_begin());

        using namespace llvm::PatternMatch;
        if (!match(
                Inst,
                m_UDiv(m_ZExtOrSelf(m_Load(m_GEP(
                           m_Intrinsic<Intrinsic::amdgcn_dispatch_ptr>(),
                           m_SpecificInt(GRID_SIZE_X + I * sizeof(uint32_t))))),
                       m_Value())))
          continue;

        IRBuilder<> Builder(Inst);

```
**EN:** This section arranges declarations into namespace scope so related AMDGPU functionality stays organized and avoids symbol collisions.
**CN:** 本节将声明组织到命名空间作用域中，使相关 AMDGPU 功能保持清晰并避免符号冲突。

### Lines 431-463: Conditional logic and checks
```cpp
        Value *GEP = Builder.CreateInBoundsGEP(
            Builder.getInt8Ty(), CI,
            {ConstantInt::get(Type::getInt64Ty(CI->getContext()),
                              HIDDEN_BLOCK_COUNT_X + I * sizeof(uint32_t))});
        Instruction *BlockCount = Builder.CreateLoad(Builder.getInt32Ty(), GEP);
        BlockCount->setMetadata(LLVMContext::MD_invariant_load,
                                MDNode::get(CI->getContext(), {}));
        BlockCount->setMetadata(LLVMContext::MD_noundef,
                                MDNode::get(CI->getContext(), {}));

        Value *BlockCountExt = Builder.CreateZExt(BlockCount, Inst->getType());
        Inst->replaceAllUsesWith(BlockCountExt);
        Inst->eraseFromParent();
        MadeChange = true;
      }
    }
  }

  // If reqd_work_group_size is set, we can replace work group size with it.
  if (!HasReqdWorkGroupSize)
    return MadeChange;

  for (int I = 0; I < 3; I++) {
    Value *GroupSize = GroupSizes[I];
    if (!GroupSize)
      continue;

    ConstantInt *KnownSize = mdconst::extract<ConstantInt>(MD->getOperand(I));
    GroupSize->replaceAllUsesWith(
        ConstantFoldIntegerCast(KnownSize, GroupSize->getType(), false, DL));
    MadeChange = true;
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `ConstantInt::get`, `Type::getInt64Ty`, `MDNode::get`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`ConstantInt::get`, `Type::getInt64Ty`, `MDNode::get`。

### Lines 464-496: Registers LLVM passes
```cpp
  return MadeChange;
}

// TODO: Move makeLIDRangeMetadata usage into here. Seem to not get
// TargetPassConfig for subtarget.
bool AMDGPULowerKernelAttributes::runOnModule(Module &M) {
  bool MadeChange = false;
  bool IsV5OrAbove =
      AMDGPU::getAMDHSACodeObjectVersion(M) >= AMDGPU::AMDHSA_COV5;
  Function *BasePtr = getBasePtrIntrinsic(M, IsV5OrAbove);

  if (!BasePtr) // ImplicitArgPtr/DispatchPtr not used.
    return false;

  SmallPtrSet<Instruction *, 4> HandledUses;
  for (auto *U : BasePtr->users()) {
    CallInst *CI = cast<CallInst>(U);
    if (HandledUses.insert(CI).second) {
      if (processUse(CI, IsV5OrAbove))
        MadeChange = true;
    }
  }

  return MadeChange;
}

INITIALIZE_PASS_BEGIN(AMDGPULowerKernelAttributes, DEBUG_TYPE,
                      "AMDGPU Kernel Attributes", false, false)
INITIALIZE_PASS_END(AMDGPULowerKernelAttributes, DEBUG_TYPE,
                    "AMDGPU Kernel Attributes", false, false)

char AMDGPULowerKernelAttributes::ID = 0;

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `AMDGPULowerKernelAttributes::runOnModule`, `AMDGPU::getAMDHSACodeObjectVersion`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`AMDGPULowerKernelAttributes::runOnModule`, `AMDGPU::getAMDHSACodeObjectVersion`。

### Lines 497-520: Implements llvm::createAMDGPULowerKernelAttributesPass
```cpp
ModulePass *llvm::createAMDGPULowerKernelAttributesPass() {
  return new AMDGPULowerKernelAttributes();
}

PreservedAnalyses
AMDGPULowerKernelAttributesPass::run(Function &F, FunctionAnalysisManager &AM) {
  bool IsV5OrAbove =
      AMDGPU::getAMDHSACodeObjectVersion(*F.getParent()) >= AMDGPU::AMDHSA_COV5;
  Function *BasePtr = getBasePtrIntrinsic(*F.getParent(), IsV5OrAbove);

  if (!BasePtr) // ImplicitArgPtr/DispatchPtr not used.
    return PreservedAnalyses::all();

  bool Changed = false;
  for (Instruction &I : instructions(F)) {
    if (CallInst *CI = dyn_cast<CallInst>(&I)) {
      if (CI->getCalledFunction() == BasePtr)
        Changed |= processUse(CI, IsV5OrAbove);
    }
  }

  return !Changed ? PreservedAnalyses::all()
                  : PreservedAnalyses::none().preserveSet<CFGAnalyses>();
}
```
**EN:** This section contains concrete logic for llvm::createAMDGPULowerKernelAttributesPass. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `llvm::createAMDGPULowerKernelAttributesPass`, `AMDGPULowerKernelAttributesPass::run`, `AMDGPU::getAMDHSACodeObjectVersion`.
**CN:** 本节包含与 llvm::createAMDGPULowerKernelAttributesPass 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`llvm::createAMDGPULowerKernelAttributesPass`, `AMDGPULowerKernelAttributesPass::run`, `AMDGPU::getAMDHSACodeObjectVersion`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `DispatchPackedOffsets`, `ImplicitArgOffsets`, `AMDGPULowerKernelAttributes`, `Intrinsic::getDeclarationIfExists`, `IsaInfo::getMaxFlatWorkGroupSize`, `ConstantInt::get`
- **Main themes / 核心主题**: instruction semantics / 指令语义; lowering / 降低; assembly handling / 汇编处理; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPU.h"`
- `"Utils/AMDGPUBaseInfo.h"`
- `"llvm/Analysis/ConstantFolding.h"`
- `"llvm/Analysis/ValueTracking.h"`
- `"llvm/CodeGen/Passes.h"`
- `"llvm/IR/Constants.h"`
- `"llvm/IR/Function.h"`
- `"llvm/IR/IRBuilder.h"`
- `"llvm/IR/InstIterator.h"`
- `"llvm/IR/Instructions.h"`
- `"llvm/IR/IntrinsicsAMDGPU.h"`
- `"llvm/IR/MDBuilder.h"`
- `"llvm/IR/PatternMatch.h"`
- `"llvm/Pass.h"`
