# AMDGPULowerIntrinsics.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPULowerIntrinsics.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPULowerIntrinsics for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPULowerIntrinsics 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33: File banner, includes, and setup
```cpp
//===-- AMDGPULowerIntrinsics.cpp -------------------------------------------=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Lower intrinsics that would otherwise require separate handling in both
// SelectionDAG and GlobalISel.
//
//===----------------------------------------------------------------------===//

#include "AMDGPU.h"
#include "AMDGPUTargetMachine.h"
#include "GCNSubtarget.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/IntrinsicInst.h"
#include "llvm/IR/IntrinsicsAMDGPU.h"
#include "llvm/InitializePasses.h"
#include "llvm/Transforms/Utils/BasicBlockUtils.h"

#define DEBUG_TYPE "amdgpu-lower-intrinsics"

using namespace llvm;

namespace {

class AMDGPULowerIntrinsicsImpl {
public:
  Module &M;
  const AMDGPUTargetMachine &TM;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `AMDGPULowerIntrinsicsImpl`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`AMDGPULowerIntrinsicsImpl`。

### Lines 34-67: Declares class AMDGPULowerIntrinsicsLegacy
```cpp
  AMDGPULowerIntrinsicsImpl(Module &M, const AMDGPUTargetMachine &TM)
      : M(M), TM(TM) {}

  bool run();

private:
  bool visitBarrier(IntrinsicInst &I);
};

class AMDGPULowerIntrinsicsLegacy : public ModulePass {
public:
  static char ID;

  AMDGPULowerIntrinsicsLegacy() : ModulePass(ID) {}

  bool runOnModule(Module &M) override;

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<TargetPassConfig>();
  }
};

template <class T> static void forEachCall(Function &Intrin, T Callback) {
  for (User *U : make_early_inc_range(Intrin.users())) {
    if (auto *CI = dyn_cast<IntrinsicInst>(U))
      Callback(CI);
  }
}

} // anonymous namespace

bool AMDGPULowerIntrinsicsImpl::run() {
  bool Changed = false;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPULowerIntrinsicsLegacy`, `AMDGPULowerIntrinsicsImpl::run`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPULowerIntrinsicsLegacy`, `AMDGPULowerIntrinsicsImpl::run`。

### Lines 68-101: Switch-based control flow
```cpp
  for (Function &F : M) {
    switch (F.getIntrinsicID()) {
    default:
      continue;
    case Intrinsic::amdgcn_s_barrier:
    case Intrinsic::amdgcn_s_barrier_signal:
    case Intrinsic::amdgcn_s_barrier_signal_isfirst:
    case Intrinsic::amdgcn_s_barrier_wait:
    case Intrinsic::amdgcn_s_cluster_barrier:
      forEachCall(F, [&](IntrinsicInst *II) { Changed |= visitBarrier(*II); });
      break;
    }
  }

  return Changed;
}

// Optimize barriers and lower s_(cluster_)barrier to a sequence of split
// barrier intrinsics.
bool AMDGPULowerIntrinsicsImpl::visitBarrier(IntrinsicInst &I) {
  assert(I.getIntrinsicID() == Intrinsic::amdgcn_s_barrier ||
         I.getIntrinsicID() == Intrinsic::amdgcn_s_barrier_signal ||
         I.getIntrinsicID() == Intrinsic::amdgcn_s_barrier_signal_isfirst ||
         I.getIntrinsicID() == Intrinsic::amdgcn_s_barrier_wait ||
         I.getIntrinsicID() == Intrinsic::amdgcn_s_cluster_barrier);

  const GCNSubtarget &ST = TM.getSubtarget<GCNSubtarget>(*I.getFunction());
  bool IsSingleWaveWG = false;

  if (TM.getOptLevel() > CodeGenOptLevel::None)
    IsSingleWaveWG = ST.isSingleWavefrontWorkgroup(*I.getFunction());

  IRBuilder<> B(&I);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPULowerIntrinsicsImpl::visitBarrier`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPULowerIntrinsicsImpl::visitBarrier`。

### Lines 102-133: Conditional logic and checks
```cpp
  // Lower the s_cluster_barrier intrinsic first. There is no corresponding
  // hardware instruction in any subtarget.
  if (I.getIntrinsicID() == Intrinsic::amdgcn_s_cluster_barrier) {
    // The default cluster barrier expects one signal per workgroup. So we need
    // a workgroup barrier first.
    if (IsSingleWaveWG) {
      B.CreateIntrinsic(B.getVoidTy(), Intrinsic::amdgcn_wave_barrier, {})
          ->copyMetadata(I);
    } else {
      Value *BarrierID_32 = B.getInt32(AMDGPU::Barrier::WORKGROUP);
      Value *BarrierID_16 = B.getInt16(AMDGPU::Barrier::WORKGROUP);
      CallInst *IsFirst = B.CreateIntrinsic(
          B.getInt1Ty(), Intrinsic::amdgcn_s_barrier_signal_isfirst,
          {BarrierID_32});
      IsFirst->copyMetadata(I);
      B.CreateIntrinsic(B.getVoidTy(), Intrinsic::amdgcn_s_barrier_wait,
                        {BarrierID_16})
          ->copyMetadata(I);

      Instruction *ThenTerm =
          SplitBlockAndInsertIfThen(IsFirst, I.getIterator(), false);
      B.SetInsertPoint(ThenTerm);
    }

    // Now we can signal the cluster barrier from a single wave and wait for the
    // barrier in all waves.
    Value *BarrierID_32 = B.getInt32(AMDGPU::Barrier::CLUSTER);
    Value *BarrierID_16 = B.getInt16(AMDGPU::Barrier::CLUSTER);
    B.CreateIntrinsic(B.getVoidTy(), Intrinsic::amdgcn_s_barrier_signal,
                      {BarrierID_32})
        ->copyMetadata(I);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 134-158: Conditional logic and checks
```cpp
    B.SetInsertPoint(&I);
    B.CreateIntrinsic(B.getVoidTy(), Intrinsic::amdgcn_s_barrier_wait,
                      {BarrierID_16})
        ->copyMetadata(I);

    I.eraseFromParent();
    return true;
  }

  bool IsWorkgroupScope = false;

  if (I.getIntrinsicID() == Intrinsic::amdgcn_s_barrier_wait ||
      I.getIntrinsicID() == Intrinsic::amdgcn_s_barrier_signal ||
      I.getIntrinsicID() == Intrinsic::amdgcn_s_barrier_signal_isfirst) {
    int BarrierID = cast<ConstantInt>(I.getArgOperand(0))->getSExtValue();
    if (BarrierID == AMDGPU::Barrier::TRAP ||
        BarrierID == AMDGPU::Barrier::WORKGROUP ||
        (BarrierID >= AMDGPU::Barrier::NAMED_BARRIER_FIRST &&
         BarrierID <= AMDGPU::Barrier::NAMED_BARRIER_LAST))
      IsWorkgroupScope = true;
  } else {
    assert(I.getIntrinsicID() == Intrinsic::amdgcn_s_barrier);
    IsWorkgroupScope = true;
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 159-191: Conditional logic and checks
```cpp
  if (IsWorkgroupScope && IsSingleWaveWG) {
    // Down-grade waits, remove split signals.
    if (I.getIntrinsicID() == Intrinsic::amdgcn_s_barrier ||
        I.getIntrinsicID() == Intrinsic::amdgcn_s_barrier_wait) {
      B.CreateIntrinsic(B.getVoidTy(), Intrinsic::amdgcn_wave_barrier, {})
          ->copyMetadata(I);
    } else if (I.getIntrinsicID() ==
               Intrinsic::amdgcn_s_barrier_signal_isfirst) {
      // If we're the only wave of the workgroup, we're always first.
      I.replaceAllUsesWith(B.getInt1(true));
    }
    I.eraseFromParent();
    return true;
  }

  if (I.getIntrinsicID() == Intrinsic::amdgcn_s_barrier &&
      ST.hasSplitBarriers()) {
    // Lower to split barriers.
    Value *BarrierID_32 = B.getInt32(AMDGPU::Barrier::WORKGROUP);
    Value *BarrierID_16 = B.getInt16(AMDGPU::Barrier::WORKGROUP);
    B.CreateIntrinsic(B.getVoidTy(), Intrinsic::amdgcn_s_barrier_signal,
                      {BarrierID_32})
        ->copyMetadata(I);
    B.CreateIntrinsic(B.getVoidTy(), Intrinsic::amdgcn_s_barrier_wait,
                      {BarrierID_16})
        ->copyMetadata(I);
    I.eraseFromParent();
    return true;
  }

  return false;
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 192-219: Preprocessor guards and macros
```cpp
PreservedAnalyses AMDGPULowerIntrinsicsPass::run(Module &M,
                                                 ModuleAnalysisManager &MAM) {
  AMDGPULowerIntrinsicsImpl Impl(M, TM);
  if (!Impl.run())
    return PreservedAnalyses::all();
  return PreservedAnalyses::none();
}

bool AMDGPULowerIntrinsicsLegacy::runOnModule(Module &M) {
  auto &TPC = getAnalysis<TargetPassConfig>();
  const AMDGPUTargetMachine &TM = TPC.getTM<AMDGPUTargetMachine>();

  AMDGPULowerIntrinsicsImpl Impl(M, TM);
  return Impl.run();
}

#define PASS_DESC "AMDGPU lower intrinsics"
INITIALIZE_PASS_BEGIN(AMDGPULowerIntrinsicsLegacy, DEBUG_TYPE, PASS_DESC, false,
                      false)
INITIALIZE_PASS_DEPENDENCY(TargetPassConfig)
INITIALIZE_PASS_END(AMDGPULowerIntrinsicsLegacy, DEBUG_TYPE, PASS_DESC, false,
                    false)

char AMDGPULowerIntrinsicsLegacy::ID = 0;

ModulePass *llvm::createAMDGPULowerIntrinsicsLegacyPass() {
  return new AMDGPULowerIntrinsicsLegacy;
}
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `AMDGPULowerIntrinsicsPass::run`, `PreservedAnalyses::all`, `PreservedAnalyses::none`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`AMDGPULowerIntrinsicsPass::run`, `PreservedAnalyses::all`, `PreservedAnalyses::none`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPULowerIntrinsicsImpl`, `AMDGPULowerIntrinsicsLegacy`, `AMDGPULowerIntrinsicsImpl::run`, `AMDGPULowerIntrinsicsImpl::visitBarrier`, `AMDGPULowerIntrinsicsPass::run`, `PreservedAnalyses::all`
- **Main themes / 核心主题**: instruction semantics / 指令语义; subtarget modeling / 子目标建模; lowering / 降低; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPU.h"`
- `"AMDGPUTargetMachine.h"`
- `"GCNSubtarget.h"`
- `"llvm/IR/IRBuilder.h"`
- `"llvm/IR/IntrinsicInst.h"`
- `"llvm/IR/IntrinsicsAMDGPU.h"`
- `"llvm/InitializePasses.h"`
- `"llvm/Transforms/Utils/BasicBlockUtils.h"`
