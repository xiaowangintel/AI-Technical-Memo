# AMDGPUImageIntrinsicOptimizer.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUImageIntrinsicOptimizer.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUImageIntrinsicOptimizer for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUImageIntrinsicOptimizer 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: File banner, license, and overview
```cpp
//===- AMDGPUImageIntrinsicOptimizer.cpp ----------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass tries to combine multiple image_load intrinsics with dim=2dmsaa
// or dim=2darraymsaa into a single image_msaa_load intrinsic if:
//
// - they refer to the same vaddr except for sample_id,
// - they use a constant sample_id and they fall into the same group,
// - they have the same dmask and the number of intrinsics and the number of
//   vaddr/vdata dword transfers is reduced by the combine.
//
// Examples for the tradeoff (all are assuming 2DMsaa for vaddr):
//
// +----------+-----+-----+-------+---------+------------+---------+----------+
// | popcount | a16 | d16 | #load | vaddr / | #msaa_load | vaddr / | combine? |
// |  (dmask) |     |     |       | vdata   |            | vdata   |          |
// +----------+-----+-----+-------+---------+------------+---------+----------+
// |        1 |   0 |   0 |     4 |  12 / 4 |          1 |   3 / 4 | yes      |
// +----------+-----+-----+-------+---------+------------+---------+----------+
// |        1 |   0 |   0 |     2 |   6 / 2 |          1 |   3 / 4 | yes?     |
// +----------+-----+-----+-------+---------+------------+---------+----------+
// |        2 |   0 |   0 |     4 |  12 / 8 |          2 |   6 / 8 | yes      |
// +----------+-----+-----+-------+---------+------------+---------+----------+
// |        2 |   0 |   0 |     2 |   6 / 4 |          2 |   6 / 8 | no       |
// +----------+-----+-----+-------+---------+------------+---------+----------+
// |        1 |   0 |   1 |     2 |   6 / 2 |          1 |   3 / 2 | yes      |
// +----------+-----+-----+-------+---------+------------+---------+----------+
//
// Some cases are of questionable benefit, like the one marked with "yes?"
```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。

### Lines 35-63: Implementation details and local logic
```cpp
// above: fewer intrinsics and fewer vaddr and fewer total transfers between SP
// and TX, but higher vdata. We start by erring on the side of converting these
// to MSAA_LOAD.
//
// clang-format off
//
// This pass will combine intrinsics such as (not neccessarily consecutive):
//  call float @llvm.amdgcn.image.load.2dmsaa.f32.i32(i32 1, i32 %s, i32 %t, i32 0, <8 x i32> %rsrc, i32 0, i32 0)
//  call float @llvm.amdgcn.image.load.2dmsaa.f32.i32(i32 1, i32 %s, i32 %t, i32 1, <8 x i32> %rsrc, i32 0, i32 0)
//  call float @llvm.amdgcn.image.load.2dmsaa.f32.i32(i32 1, i32 %s, i32 %t, i32 2, <8 x i32> %rsrc, i32 0, i32 0)
//  call float @llvm.amdgcn.image.load.2dmsaa.f32.i32(i32 1, i32 %s, i32 %t, i32 3, <8 x i32> %rsrc, i32 0, i32 0)
// ==>
//  call <4 x float> @llvm.amdgcn.image.msaa.load.2dmsaa.v4f32.i32(i32 1, i32 %s, i32 %t, i32 0, <8 x i32> %rsrc, i32 0, i32 0)
//
// clang-format on
//
// Future improvements:
//
// - We may occasionally not want to do the combine if it increases the maximum
//   register pressure.
//
// - Ensure clausing when multiple MSAA_LOAD are generated.
//
// Note: Even though the image_msaa_load intrinsic already exists on gfx10, this
// combine only applies to gfx11, due to a limitation in gfx10: the gfx10
// IMAGE_MSAA_LOAD only works correctly with single-channel texture formats, and
// we don't know the format at compile time.
//===----------------------------------------------------------------------===//

```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 64-97: Header dependencies and setup
```cpp
#include "AMDGPU.h"
#include "AMDGPUInstrInfo.h"
#include "AMDGPUTargetMachine.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/IntrinsicInst.h"
#include "llvm/IR/IntrinsicsAMDGPU.h"
#include "llvm/Pass.h"
#include "llvm/Support/raw_ostream.h"

using namespace llvm;

#define DEBUG_TYPE "amdgpu-image-intrinsic-opt"

namespace {
class AMDGPUImageIntrinsicOptimizer : public FunctionPass {
  const TargetMachine *TM;

public:
  static char ID;

  AMDGPUImageIntrinsicOptimizer(const TargetMachine *TM = nullptr)
      : FunctionPass(ID), TM(TM) {}

  bool runOnFunction(Function &F) override;

}; // End of class AMDGPUImageIntrinsicOptimizer
} // End anonymous namespace

INITIALIZE_PASS(AMDGPUImageIntrinsicOptimizer, DEBUG_TYPE,
                "AMDGPU Image Intrinsic Optimizer", false, false)

char AMDGPUImageIntrinsicOptimizer::ID = 0;

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `AMDGPUImageIntrinsicOptimizer`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`AMDGPUImageIntrinsicOptimizer`。

### Lines 98-129: Defines addInstToMergeableList
```cpp
void addInstToMergeableList(
    IntrinsicInst *II,
    SmallVector<SmallVector<IntrinsicInst *, 4>> &MergeableInsts,
    const AMDGPU::ImageDimIntrinsicInfo *ImageDimIntr) {
  for (SmallVector<IntrinsicInst *, 4> &IIList : MergeableInsts) {
    // Check Dim.
    if (IIList.front()->getIntrinsicID() != II->getIntrinsicID())
      continue;

    // Check D16.
    if (IIList.front()->getType() != II->getType())
      continue;

    // Check all arguments (DMask, VAddr, RSrc etc).
    bool AllEqual = true;
    assert(IIList.front()->arg_size() == II->arg_size());
    for (int I = 1, E = II->arg_size(); AllEqual && I != E; ++I) {
      Value *ArgList = IIList.front()->getArgOperand(I);
      Value *Arg = II->getArgOperand(I);
      if (I == ImageDimIntr->VAddrEnd - 1) {
        // Check FragId group.
        auto *FragIdList = cast<ConstantInt>(IIList.front()->getArgOperand(I));
        auto *FragId = cast<ConstantInt>(II->getArgOperand(I));
        AllEqual = FragIdList->getValue().udiv(4) == FragId->getValue().udiv(4);
      } else {
        // Check all arguments except FragId.
        AllEqual = ArgList == Arg;
      }
    }
    if (!AllEqual)
      continue;

```
**EN:** This section contains concrete logic for addInstToMergeableList. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 addInstToMergeableList 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 130-161: Declares emplace_back
```cpp
    // Add to the list.
    IIList.emplace_back(II);
    return;
  }

  // Similar instruction not found, so add a new list.
  MergeableInsts.emplace_back(1, II);
  LLVM_DEBUG(dbgs() << "New: " << *II << "\n");
}

// Collect list of all instructions we know how to merge in a subset of the
// block. It returns an iterator to the instruction after the last one analyzed.
BasicBlock::iterator collectMergeableInsts(
    BasicBlock::iterator I, BasicBlock::iterator E,
    SmallVector<SmallVector<IntrinsicInst *, 4>> &MergeableInsts) {
  for (; I != E; ++I) {
    // Don't combine if there is a store in the middle or if there is a memory
    // barrier.
    if (I->mayHaveSideEffects()) {
      ++I;
      break;
    }

    // Ignore non-intrinsics.
    if (IntrinsicInst *II = dyn_cast<IntrinsicInst>(I)) {
      Intrinsic::ID IntrinID = II->getIntrinsicID();

      // Ignore other intrinsics.
      if (IntrinID != Intrinsic::amdgcn_image_load_2dmsaa &&
          IntrinID != Intrinsic::amdgcn_image_load_2darraymsaa)
        continue;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 162-193: Conditional logic and checks
```cpp
      // Check for constant FragId.
      const auto *ImageDimIntr = AMDGPU::getImageDimIntrinsicInfo(IntrinID);
      const uint8_t FragIdIndex = ImageDimIntr->VAddrEnd - 1;
      if (!isa<ConstantInt>(II->getArgOperand(FragIdIndex)))
        continue;

      LLVM_DEBUG(dbgs() << "Merge: " << *II << "\n");
      addInstToMergeableList(II, MergeableInsts, ImageDimIntr);
    }
  }

  return I;
}

bool optimizeSection(ArrayRef<SmallVector<IntrinsicInst *, 4>> MergeableInsts) {
  bool Modified = false;

  SmallVector<Instruction *, 4> InstrsToErase;
  for (const auto &IIList : MergeableInsts) {
    if (IIList.size() <= 1)
      continue;

    // Assume the arguments are unchanged and later override them, if needed.
    SmallVector<Value *, 16> Args(IIList.front()->args());

    // Validate function argument and return types, extracting overloaded
    // types along the way.
    SmallVector<Type *, 6> OverloadTys;
    Function *F = IIList.front()->getCalledFunction();
    if (!Intrinsic::isSignatureValid(F, OverloadTys))
      continue;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::getImageDimIntrinsicInfo`, `Intrinsic::isSignatureValid`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::getImageDimIntrinsicInfo`, `Intrinsic::isSignatureValid`。

### Lines 194-225: Conditional logic and checks
```cpp
    Intrinsic::ID IntrinID = IIList.front()->getIntrinsicID();
    const AMDGPU::ImageDimIntrinsicInfo *ImageDimIntr =
        AMDGPU::getImageDimIntrinsicInfo(IntrinID);

    Type *EltTy = IIList.front()->getType()->getScalarType();
    Type *NewTy = FixedVectorType::get(EltTy, 4);
    OverloadTys[0] = NewTy;
    bool isD16 = EltTy->isHalfTy();

    ConstantInt *DMask = cast<ConstantInt>(
        IIList.front()->getArgOperand(ImageDimIntr->DMaskIndex));
    unsigned DMaskVal = DMask->getZExtValue() & 0xf;
    unsigned NumElts = popcount(DMaskVal);

    // Number of instructions and the number of vaddr/vdata dword transfers
    // should be reduced.
    unsigned NumLoads = IIList.size();
    unsigned NumMsaas = NumElts;
    unsigned NumVAddrLoads = 3 * NumLoads;
    unsigned NumVDataLoads = divideCeil(NumElts, isD16 ? 2 : 1) * NumLoads;
    unsigned NumVAddrMsaas = 3 * NumMsaas;
    unsigned NumVDataMsaas = divideCeil(4, isD16 ? 2 : 1) * NumMsaas;

    if (NumLoads < NumMsaas ||
        (NumVAddrLoads + NumVDataLoads < NumVAddrMsaas + NumVDataMsaas))
      continue;

    const uint8_t FragIdIndex = ImageDimIntr->VAddrEnd - 1;
    auto *FragId =
        cast<ConstantInt>(IIList.front()->getArgOperand(FragIdIndex));
    const APInt &NewFragIdVal = FragId->getValue().udiv(4) * 4;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::getImageDimIntrinsicInfo`, `FixedVectorType::get`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::getImageDimIntrinsicInfo`, `FixedVectorType::get`。

### Lines 226-249: Conditional logic and checks
```cpp
    // Create the new instructions.
    IRBuilder<> B(IIList.front());

    // Create the new image_msaa_load intrinsic.
    SmallVector<Instruction *, 4> NewCalls;
    while (DMaskVal != 0) {
      unsigned NewMaskVal = 1 << countr_zero(DMaskVal);

      Intrinsic::ID NewIntrinID;
      if (IntrinID == Intrinsic::amdgcn_image_load_2dmsaa)
        NewIntrinID = Intrinsic::amdgcn_image_msaa_load_2dmsaa;
      else
        NewIntrinID = Intrinsic::amdgcn_image_msaa_load_2darraymsaa;

      Args[ImageDimIntr->DMaskIndex] =
          ConstantInt::get(DMask->getType(), NewMaskVal);
      Args[FragIdIndex] = ConstantInt::get(FragId->getType(), NewFragIdVal);
      CallInst *NewCall = B.CreateIntrinsic(NewIntrinID, OverloadTys, Args);
      LLVM_DEBUG(dbgs() << "Optimize: " << *NewCall << "\n");

      NewCalls.push_back(NewCall);
      DMaskVal -= NewMaskVal;
    }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `ConstantInt::get`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`ConstantInt::get`。

### Lines 250-282: Conditional logic and checks
```cpp
    // Create the new extractelement instructions.
    for (auto &II : IIList) {
      Value *VecOp = nullptr;
      auto *Idx = cast<ConstantInt>(II->getArgOperand(FragIdIndex));
      B.SetCurrentDebugLocation(II->getDebugLoc());
      if (NumElts == 1) {
        VecOp = B.CreateExtractElement(NewCalls[0], Idx->getValue().urem(4));
        LLVM_DEBUG(dbgs() << "Add: " << *VecOp << "\n");
      } else {
        VecOp = PoisonValue::get(II->getType());
        for (unsigned I = 0; I < NumElts; ++I) {
          VecOp = B.CreateInsertElement(
              VecOp,
              B.CreateExtractElement(NewCalls[I], Idx->getValue().urem(4)), I);
          LLVM_DEBUG(dbgs() << "Add: " << *VecOp << "\n");
        }
      }

      // Replace the old instruction.
      II->replaceAllUsesWith(VecOp);
      VecOp->takeName(II);
      InstrsToErase.push_back(II);
    }

    Modified = true;
  }

  for (auto *I : InstrsToErase)
    I->eraseFromParent();

  return Modified;
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `PoisonValue::get`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`PoisonValue::get`。

### Lines 283-316: Defines imageIntrinsicOptimizerImpl
```cpp
static bool imageIntrinsicOptimizerImpl(Function &F, const TargetMachine *TM) {
  if (!TM)
    return false;

  // This optimization only applies to GFX11 and beyond.
  const GCNSubtarget &ST = TM->getSubtarget<GCNSubtarget>(F);
  if (!AMDGPU::isGFX11Plus(ST) || ST.hasMSAALoadDstSelBug())
    return false;

  Module *M = F.getParent();

  // Early test to determine if the intrinsics are used.
  if (llvm::none_of(*M, [](Function &F) {
        return !F.users().empty() &&
               (F.getIntrinsicID() == Intrinsic::amdgcn_image_load_2dmsaa ||
                F.getIntrinsicID() == Intrinsic::amdgcn_image_load_2darraymsaa);
      }))
    return false;

  bool Modified = false;
  for (auto &BB : F) {
    BasicBlock::iterator SectionEnd;
    for (BasicBlock::iterator I = BB.begin(), E = BB.end(); I != E;
         I = SectionEnd) {
      SmallVector<SmallVector<IntrinsicInst *, 4>> MergeableInsts;

      SectionEnd = collectMergeableInsts(I, E, MergeableInsts);
      Modified |= optimizeSection(MergeableInsts);
    }
  }

  return Modified;
}

```
**EN:** This section contains concrete logic for imageIntrinsicOptimizerImpl. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPU::isGFX11Plus`, `llvm::none_of`.
**CN:** 本节包含与 imageIntrinsicOptimizerImpl 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPU::isGFX11Plus`, `llvm::none_of`。

### Lines 317-335: Implements AMDGPUImageIntrinsicOptimizer::runOnFunction
```cpp
bool AMDGPUImageIntrinsicOptimizer::runOnFunction(Function &F) {
  if (skipFunction(F))
    return false;

  return imageIntrinsicOptimizerImpl(F, TM);
}

FunctionPass *
llvm::createAMDGPUImageIntrinsicOptimizerPass(const TargetMachine *TM) {
  return new AMDGPUImageIntrinsicOptimizer(TM);
}

PreservedAnalyses
AMDGPUImageIntrinsicOptimizerPass::run(Function &F,
                                       FunctionAnalysisManager &AM) {

  bool Changed = imageIntrinsicOptimizerImpl(F, &TM);
  return Changed ? PreservedAnalyses::none() : PreservedAnalyses::all();
}
```
**EN:** This section contains concrete logic for AMDGPUImageIntrinsicOptimizer::runOnFunction. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUImageIntrinsicOptimizer::runOnFunction`, `llvm::createAMDGPUImageIntrinsicOptimizerPass`, `AMDGPUImageIntrinsicOptimizerPass::run`.
**CN:** 本节包含与 AMDGPUImageIntrinsicOptimizer::runOnFunction 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUImageIntrinsicOptimizer::runOnFunction`, `llvm::createAMDGPUImageIntrinsicOptimizerPass`, `AMDGPUImageIntrinsicOptimizerPass::run`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPUImageIntrinsicOptimizer`, `AMDGPU::getImageDimIntrinsicInfo`, `Intrinsic::isSignatureValid`, `FixedVectorType::get`, `ConstantInt::get`, `PoisonValue::get`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPU.h"`
- `"AMDGPUInstrInfo.h"`
- `"AMDGPUTargetMachine.h"`
- `"llvm/IR/Function.h"`
- `"llvm/IR/IRBuilder.h"`
- `"llvm/IR/IntrinsicInst.h"`
- `"llvm/IR/IntrinsicsAMDGPU.h"`
- `"llvm/Pass.h"`
- `"llvm/Support/raw_ostream.h"`
