# AMDGPURemoveIncompatibleFunctions.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPURemoveIncompatibleFunctions.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPURemoveIncompatibleFunctions for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPURemoveIncompatibleFunctions 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: File banner, includes, and setup
```cpp
//===-- AMDGPURemoveIncompatibleFunctions.cpp -----------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// This pass replaces all uses of functions that use GPU features
/// incompatible with the current GPU with null then deletes the function.
//
//===----------------------------------------------------------------------===//

#include "AMDGPURemoveIncompatibleFunctions.h"
#include "AMDGPU.h"
#include "GCNSubtarget.h"
#include "llvm/Analysis/OptimizationRemarkEmitter.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/Module.h"
#include "llvm/Pass.h"
#include "llvm/Target/TargetMachine.h"

#define DEBUG_TYPE "amdgpu-remove-incompatible-functions"

using namespace llvm;

namespace llvm {
extern const SubtargetFeatureKV
    AMDGPUFeatureKV[AMDGPU::NumSubtargetFeatures - 1];
} // namespace llvm

namespace {

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 35-65: Declares class AMDGPURemoveIncompatibleFunctions
```cpp
using Generation = AMDGPUSubtarget::Generation;

class AMDGPURemoveIncompatibleFunctions {
public:
  AMDGPURemoveIncompatibleFunctions(const TargetMachine *TM = nullptr)
      : TM(TM) {
    assert(TM && "No TargetMachine!");
  }
  /// Checks a single function, returns true if the function must be deleted.
  bool checkFunction(Function &F);

  bool run(Module &M) {
    assert(TM->getTargetTriple().isAMDGCN());

    SmallVector<Function *, 4> FnsToDelete;
    for (Function &F : M) {
      if (checkFunction(F))
        FnsToDelete.push_back(&F);
    }

    for (Function *F : FnsToDelete) {
      F->replaceAllUsesWith(ConstantPointerNull::get(F->getType()));
      F->eraseFromParent();
    }
    return !FnsToDelete.empty();
  }

private:
  const TargetMachine *TM = nullptr;
};

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPURemoveIncompatibleFunctions`, `ConstantPointerNull::get`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPURemoveIncompatibleFunctions`, `ConstantPointerNull::get`。

### Lines 66-95: Declares class AMDGPURemoveIncompatibleFunctionsLegacy
```cpp
class AMDGPURemoveIncompatibleFunctionsLegacy : public ModulePass {
public:
  static char ID;

  AMDGPURemoveIncompatibleFunctionsLegacy(const TargetMachine *TM)
      : ModulePass(ID), TM(TM) {}

  bool runOnModule(Module &M) override {
    AMDGPURemoveIncompatibleFunctions Pass(TM);
    return Pass.run(M);
  }

  StringRef getPassName() const override {
    return "AMDGPU Remove Incompatible Functions";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {}

private:
  const TargetMachine *TM = nullptr;
};

StringRef getFeatureName(unsigned Feature) {
  for (const SubtargetFeatureKV &KV : AMDGPUFeatureKV)
    if (Feature == KV.Value)
      return KV.Key;

  llvm_unreachable("Unknown Target feature");
}

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPURemoveIncompatibleFunctionsLegacy`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPURemoveIncompatibleFunctionsLegacy`。

### Lines 96-125: Defines getGPUInfo
```cpp
const SubtargetSubTypeKV *getGPUInfo(const GCNSubtarget &ST,
                                     StringRef GPUName) {
  for (const SubtargetSubTypeKV &KV : ST.getAllProcessorDescriptions())
    if (StringRef(KV.Key) == GPUName)
      return &KV;

  return nullptr;
}

constexpr unsigned FeaturesToCheck[] = {AMDGPU::FeatureGFX11Insts,
                                        AMDGPU::FeatureGFX10Insts,
                                        AMDGPU::FeatureGFX9Insts,
                                        AMDGPU::FeatureGFX8Insts,
                                        AMDGPU::FeatureDPP,
                                        AMDGPU::FeatureDPPWavefrontShifts,
                                        AMDGPU::FeatureDPPBroadcasts,
                                        AMDGPU::Feature16BitInsts,
                                        AMDGPU::FeatureDot1Insts,
                                        AMDGPU::FeatureDot2Insts,
                                        AMDGPU::FeatureDot3Insts,
                                        AMDGPU::FeatureDot4Insts,
                                        AMDGPU::FeatureDot5Insts,
                                        AMDGPU::FeatureDot6Insts,
                                        AMDGPU::FeatureDot7Insts,
                                        AMDGPU::FeatureDot8Insts,
                                        AMDGPU::FeatureExtendedImageInsts,
                                        AMDGPU::FeatureSMemRealTime,
                                        AMDGPU::FeatureSMemTimeInst,
                                        AMDGPU::FeatureGWS};

```
**EN:** This section contains concrete logic for getGPUInfo. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getGPUInfo 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 126-158: Defines expandImpliedFeatures
```cpp
FeatureBitset expandImpliedFeatures(const FeatureBitset &Features) {
  FeatureBitset Result = Features;
  for (const SubtargetFeatureKV &FE : AMDGPUFeatureKV) {
    if (Features.test(FE.Value) && FE.Implies.any())
      Result |= expandImpliedFeatures(FE.Implies.getAsBitset());
  }
  return Result;
}

void reportFunctionRemoved(Function &F, unsigned Feature) {
  OptimizationRemarkEmitter ORE(&F);
  ORE.emit([&]() {
    // Note: we print the function name as part of the diagnostic because if
    // debug info is not present, users get "<unknown>:0:0" as the debug
    // loc. If we didn't print the function name there would be no way to
    // tell which function got removed.
    return OptimizationRemark(DEBUG_TYPE, "AMDGPUIncompatibleFnRemoved", &F)
           << "removing function '" << F.getName() << "': +"
           << getFeatureName(Feature)
           << " is not supported on the current target";
  });
}
} // end anonymous namespace

PreservedAnalyses
AMDGPURemoveIncompatibleFunctionsPass::run(Module &M,
                                           ModuleAnalysisManager &MAM) {
  AMDGPURemoveIncompatibleFunctions Impl(TM);
  if (Impl.run(M))
    return PreservedAnalyses::none();
  return PreservedAnalyses::all();
}

```
**EN:** This section contains concrete logic for expandImpliedFeatures. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPURemoveIncompatibleFunctionsPass::run`, `PreservedAnalyses::none`, `PreservedAnalyses::all`.
**CN:** 本节包含与 expandImpliedFeatures 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPURemoveIncompatibleFunctionsPass::run`, `PreservedAnalyses::none`, `PreservedAnalyses::all`。

### Lines 159-188: Implements AMDGPURemoveIncompatibleFunctions::checkFunction
```cpp
bool AMDGPURemoveIncompatibleFunctions::checkFunction(Function &F) {
  if (F.isDeclaration())
    return false;

  const GCNSubtarget *ST =
      static_cast<const GCNSubtarget *>(TM->getSubtargetImpl(F));

  // Check the GPU isn't generic or generic-hsa. Generic is used for testing
  // only and we don't want this pass to interfere with it.
  StringRef GPUName = ST->getCPU();
  if (GPUName.empty() || GPUName.starts_with("generic"))
    return false;

  // Try to fetch the GPU's info. If we can't, it's likely an unknown processor
  // so just bail out.
  const SubtargetSubTypeKV *GPUInfo = getGPUInfo(*ST, GPUName);
  if (!GPUInfo)
    return false;

  // Get all the features implied by the current GPU, and recursively expand
  // the features that imply other features.
  //
  // e.g. GFX90A implies FeatureGFX9, and FeatureGFX9 implies a whole set of
  // other features.
  const FeatureBitset GPUFeatureBits =
      expandImpliedFeatures(GPUInfo->Implies.getAsBitset());

  // Now that the have a FeatureBitset containing all possible features for
  // the chosen GPU, check our list of "suspicious" features.

```
**EN:** This section contains concrete logic for AMDGPURemoveIncompatibleFunctions::checkFunction. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPURemoveIncompatibleFunctions::checkFunction`.
**CN:** 本节包含与 AMDGPURemoveIncompatibleFunctions::checkFunction 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPURemoveIncompatibleFunctions::checkFunction`。

### Lines 189-222: Registers LLVM passes
```cpp
  // Check that the user didn't enable any features that aren't part of that
  // GPU's feature set. We only check a predetermined set of features.
  for (unsigned Feature : FeaturesToCheck) {
    if (ST->hasFeature(Feature) && !GPUFeatureBits.test(Feature)) {
      reportFunctionRemoved(F, Feature);
      return true;
    }
  }

  // Delete FeatureWavefrontSize32 functions for
  // gfx9 and below targets that don't support the mode.
  // gfx10, gfx11, gfx12 are implied to support both wave32 and 64 features.
  // They are not in the feature set. So, we need a separate check
  if (!ST->supportsWave32() && ST->hasFeature(AMDGPU::FeatureWavefrontSize32)) {
    reportFunctionRemoved(F, AMDGPU::FeatureWavefrontSize32);
    return true;
  }
  // gfx125x only support FeatureWavefrontSize32.
  if (!ST->supportsWave64() && ST->hasFeature(AMDGPU::FeatureWavefrontSize64)) {
    reportFunctionRemoved(F, AMDGPU::FeatureWavefrontSize64);
    return true;
  }
  return false;
}

INITIALIZE_PASS(AMDGPURemoveIncompatibleFunctionsLegacy, DEBUG_TYPE,
                "AMDGPU Remove Incompatible Functions", false, false)

char AMDGPURemoveIncompatibleFunctionsLegacy::ID = 0;

ModulePass *
llvm::createAMDGPURemoveIncompatibleFunctionsPass(const TargetMachine *TM) {
  return new AMDGPURemoveIncompatibleFunctionsLegacy(TM);
}
```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `llvm::createAMDGPURemoveIncompatibleFunctionsPass`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`llvm::createAMDGPURemoveIncompatibleFunctionsPass`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPURemoveIncompatibleFunctions`, `AMDGPURemoveIncompatibleFunctionsLegacy`, `ConstantPointerNull::get`, `AMDGPURemoveIncompatibleFunctionsPass::run`, `PreservedAnalyses::none`, `PreservedAnalyses::all`
- **Main themes / 核心主题**: subtarget modeling / 子目标建模; feature description / 特性描述; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPURemoveIncompatibleFunctions.h"`
- `"AMDGPU.h"`
- `"GCNSubtarget.h"`
- `"llvm/Analysis/OptimizationRemarkEmitter.h"`
- `"llvm/IR/Function.h"`
- `"llvm/IR/Module.h"`
- `"llvm/Pass.h"`
- `"llvm/Target/TargetMachine.h"`
