# AArch64O0PreLegalizerCombiner.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/GISel/AArch64O0PreLegalizerCombiner.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This file implements type and operation legalization for the AArch64 backend. / 该文件实现 AArch64 后端中的类型与操作合法化。
## Line-by-Line Analysis / 逐行分析
### Lines 1-28: Documented code section
```cpp
//=== lib/CodeGen/GlobalISel/AArch64O0PreLegalizerCombiner.cpp ------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass does combining of machine instructions at the generic MI level,
// before the legalizer.
//
//===----------------------------------------------------------------------===//

#include "AArch64.h"
#include "AArch64GlobalISelUtils.h"
#include "AArch64TargetMachine.h"
#include "llvm/CodeGen/GlobalISel/Combiner.h"
#include "llvm/CodeGen/GlobalISel/CombinerHelper.h"
#include "llvm/CodeGen/GlobalISel/CombinerInfo.h"
#include "llvm/CodeGen/GlobalISel/GIMatchTableExecutorImpl.h"
#include "llvm/CodeGen/GlobalISel/GISelValueTracking.h"
#include "llvm/CodeGen/GlobalISel/MIPatternMatch.h"
#include "llvm/CodeGen/GlobalISel/MachineIRBuilder.h"
#include "llvm/CodeGen/LibcallLoweringInfo.h"
#include "llvm/CodeGen/MachineDominators.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionAnalysisManager.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 29-54: Includes and compile-time dependencies
```cpp
#include "llvm/CodeGen/MachinePassManager.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/IR/Instructions.h"
#include <memory>

#define GET_GICOMBINER_DEPS
#include "AArch64GenO0PreLegalizeGICombiner.inc"
#undef GET_GICOMBINER_DEPS

#define DEBUG_TYPE "aarch64-O0-prelegalizer-combiner"

using namespace llvm;
using namespace MIPatternMatch;

#define GET_GICOMBINER_TYPES
#include "AArch64GenO0PreLegalizeGICombiner.inc"
#undef GET_GICOMBINER_TYPES

namespace {

class AArch64O0PreLegalizerCombinerImpl : public Combiner {
protected:
  const CombinerHelper Helper;
  const AArch64O0PreLegalizerCombinerImplRuleConfig &RuleConfig;
  const AArch64Subtarget &STI;
  const LibcallLoweringInfo &Libcalls;
```
**EN:** This block pulls in the target-specific headers and LLVM infrastructure needed by the rest of the file.  
**CN:** 该代码块引入后续实现所需的目标专用头文件与 LLVM 基础设施。
### Lines 55-76: Function getName
```cpp

public:
  AArch64O0PreLegalizerCombinerImpl(
      MachineFunction &MF, CombinerInfo &CInfo, GISelCSEInfo *CSEInfo,
      const AArch64O0PreLegalizerCombinerImplRuleConfig &RuleConfig,
      const AArch64Subtarget &STI, const LibcallLoweringInfo &Libcalls);

  static const char *getName() { return "AArch64O0PreLegalizerCombiner"; }

  bool tryCombineAll(MachineInstr &I) const override;

  bool tryCombineAllImpl(MachineInstr &I) const;

private:
#define GET_GICOMBINER_CLASS_MEMBERS
#include "AArch64GenO0PreLegalizeGICombiner.inc"
#undef GET_GICOMBINER_CLASS_MEMBERS
};

#define GET_GICOMBINER_IMPL
#include "AArch64GenO0PreLegalizeGICombiner.inc"
#undef GET_GICOMBINER_IMPL
```
**EN:** This block implements getName, advancing the file's type and operation legalization flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 getName，通过 AArch64 专用的决策与数据处理推进本文件的类型与操作合法化流程。
### Lines 77-93: Core AArch64 backend logic
```cpp

AArch64O0PreLegalizerCombinerImpl::AArch64O0PreLegalizerCombinerImpl(
    MachineFunction &MF, CombinerInfo &CInfo, GISelCSEInfo *CSEInfo,
    const AArch64O0PreLegalizerCombinerImplRuleConfig &RuleConfig,
    const AArch64Subtarget &STI, const LibcallLoweringInfo &Libcalls)
    : Combiner(MF, CInfo, /*VT=*/nullptr, CSEInfo),
      Helper(Observer, B, /*IsPreLegalize*/ true, /*VT=*/nullptr),
      RuleConfig(RuleConfig), STI(STI), Libcalls(Libcalls),
#define GET_GICOMBINER_CONSTRUCTOR_INITS
#include "AArch64GenO0PreLegalizeGICombiner.inc"
#undef GET_GICOMBINER_CONSTRUCTOR_INITS
{
}

bool AArch64O0PreLegalizerCombinerImpl::tryCombineAll(MachineInstr &MI) const {
  if (tryCombineAllImpl(MI))
    return true;
```
**EN:** This block continues the file's main type and operation legalization logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的类型与操作合法化主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 94-117: Core AArch64 backend logic
```cpp

  unsigned Opc = MI.getOpcode();
  switch (Opc) {
  case TargetOpcode::G_SHUFFLE_VECTOR:
    return Helper.tryCombineShuffleVector(MI);
  case TargetOpcode::G_MEMCPY_INLINE:
    return Helper.tryEmitMemcpyInline(MI);
  case TargetOpcode::G_MEMCPY:
  case TargetOpcode::G_MEMMOVE:
  case TargetOpcode::G_MEMSET: {
    // At -O0 set a maxlen of 32 to inline;
    unsigned MaxLen = 32;
    // Try to inline memcpy type calls if optimizations are enabled.
    if (Helper.tryCombineMemCpyFamily(MI, MaxLen))
      return true;
    if (Opc == TargetOpcode::G_MEMSET)
      return llvm::AArch64GISelUtils::tryEmitBZero(MI, B, Libcalls,
                                                   CInfo.EnableMinSize);
    return false;
  }
  }

  return false;
}
```
**EN:** This block continues the file's main type and operation legalization logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的类型与操作合法化主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 118-143: Function runCombiner
```cpp

bool runCombiner(
    MachineFunction &MF, const LibcallLoweringInfo &Libcalls,
    const AArch64O0PreLegalizerCombinerImplRuleConfig &RuleConfig) {
  const Function &F = MF.getFunction();
  const AArch64Subtarget &ST = MF.getSubtarget<AArch64Subtarget>();

  CombinerInfo CInfo(/*AllowIllegalOps=*/true, /*ShouldLegalizeIllegal=*/false,
                     /*LegalizerInfo=*/nullptr, /*EnableOpt=*/false,
                     F.hasOptSize(), F.hasMinSize());
  // Disable fixed-point iteration in the Combiner. This improves compile-time
  // at the cost of possibly missing optimizations. See PR#94291 for details.
  CInfo.MaxIterations = 1;

  AArch64O0PreLegalizerCombinerImpl Impl(MF, CInfo,
                                         /*CSEInfo*/ nullptr, RuleConfig, ST,
                                         Libcalls);
  return Impl.combineMachineInstrs();
}

// Pass boilerplate
// ================

class AArch64O0PreLegalizerCombinerLegacy : public MachineFunctionPass {
public:
  static char ID;
```
**EN:** This block implements runCombiner, advancing the file's type and operation legalization flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 runCombiner，通过 AArch64 专用的决策与数据处理推进本文件的类型与操作合法化流程。
### Lines 144-166: Function getPassName
```cpp

  AArch64O0PreLegalizerCombinerLegacy();

  StringRef getPassName() const override {
    return "AArch64O0PreLegalizerCombiner";
  }

  bool runOnMachineFunction(MachineFunction &MF) override;

  void getAnalysisUsage(AnalysisUsage &AU) const override;

private:
  AArch64O0PreLegalizerCombinerImplRuleConfig RuleConfig;
};
} // end anonymous namespace

void AArch64O0PreLegalizerCombinerLegacy::getAnalysisUsage(
    AnalysisUsage &AU) const {
  AU.setPreservesCFG();
  getSelectionDAGFallbackAnalysisUsage(AU);
  AU.addRequired<LibcallLoweringInfoWrapper>();
  MachineFunctionPass::getAnalysisUsage(AU);
}
```
**EN:** This block implements getPassName, advancing the file's type and operation legalization flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 getPassName，通过 AArch64 专用的决策与数据处理推进本文件的类型与操作合法化流程。
### Lines 167-187: Function AArch64O0PreLegalizerCombinerLegacy::AArch64O0PreLegalizerCombinerLegacy
```cpp

AArch64O0PreLegalizerCombinerLegacy::AArch64O0PreLegalizerCombinerLegacy()
    : MachineFunctionPass(ID) {
  if (!RuleConfig.parseCommandLineOption())
    report_fatal_error("Invalid rule identifier");
}

bool AArch64O0PreLegalizerCombinerLegacy::runOnMachineFunction(
    MachineFunction &MF) {
  if (MF.getProperties().hasFailedISel())
    return false;

  const Function &F = MF.getFunction();

  const AArch64Subtarget &ST = MF.getSubtarget<AArch64Subtarget>();
  const LibcallLoweringInfo &Libcalls =
      getAnalysis<LibcallLoweringInfoWrapper>().getLibcallLowering(
          *F.getParent(), ST);

  return runCombiner(MF, Libcalls, RuleConfig);
}
```
**EN:** This block implements AArch64O0PreLegalizerCombinerLegacy::AArch64O0PreLegalizerCombinerLegacy, advancing the file's type and operation legalization flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 AArch64O0PreLegalizerCombinerLegacy::AArch64O0PreLegalizerCombinerLegacy，通过 AArch64 专用的决策与数据处理推进本文件的类型与操作合法化流程。
### Lines 188-210: Pass registration macros
```cpp

char AArch64O0PreLegalizerCombinerLegacy::ID = 0;
INITIALIZE_PASS_BEGIN(AArch64O0PreLegalizerCombinerLegacy, DEBUG_TYPE,
                      "Combine AArch64 machine instrs before legalization",
                      false, false)
INITIALIZE_PASS_DEPENDENCY(GISelCSEAnalysisWrapperPass)
INITIALIZE_PASS_DEPENDENCY(LibcallLoweringInfoWrapper)
INITIALIZE_PASS_END(AArch64O0PreLegalizerCombinerLegacy, DEBUG_TYPE,
                    "Combine AArch64 machine instrs before legalization", false,
                    false)

AArch64O0PreLegalizerCombinerPass::AArch64O0PreLegalizerCombinerPass()
    : RuleConfig(
          std::make_unique<AArch64O0PreLegalizerCombinerImplRuleConfig>()) {
  if (!RuleConfig->parseCommandLineOption())
    report_fatal_error("Invalid rule identifier");
}

AArch64O0PreLegalizerCombinerPass::AArch64O0PreLegalizerCombinerPass(
    AArch64O0PreLegalizerCombinerPass &&) = default;

AArch64O0PreLegalizerCombinerPass::~AArch64O0PreLegalizerCombinerPass() =
    default;
```
**EN:** This block registers the pass with LLVM so the surrounding transformation can be constructed and scheduled by name.  
**CN:** 该代码块把该 pass 注册到 LLVM 中，使外围变换能够按名称构建并加入流水线。
### Lines 211-235: Function AArch64O0PreLegalizerCombinerPass::run
```cpp

PreservedAnalyses
AArch64O0PreLegalizerCombinerPass::run(MachineFunction &MF,
                                       MachineFunctionAnalysisManager &MFAM) {
  if (MF.getProperties().hasFailedISel())
    return PreservedAnalyses::all();

  const AArch64Subtarget &ST = MF.getSubtarget<AArch64Subtarget>();
  auto &MAMProxy =
      MFAM.getResult<ModuleAnalysisManagerMachineFunctionProxy>(MF);
  const LibcallLoweringModuleAnalysisResult *LibcallResult =
      MAMProxy.getCachedResult<LibcallLoweringModuleAnalysis>(
          *MF.getFunction().getParent());
  if (!LibcallResult)
    reportFatalUsageError("LibcallLoweringModuleAnalysis result not available");

  const LibcallLoweringInfo &Libcalls = LibcallResult->getLibcallLowering(ST);

  if (!runCombiner(MF, Libcalls, *RuleConfig))
    return PreservedAnalyses::all();

  PreservedAnalyses PA = getMachineFunctionPassPreservedAnalyses();
  PA.preserveSet<CFGAnalyses>();
  return PA;
}
```
**EN:** This block implements AArch64O0PreLegalizerCombinerPass::run, advancing the file's type and operation legalization flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 AArch64O0PreLegalizerCombinerPass::run，通过 AArch64 专用的决策与数据处理推进本文件的类型与操作合法化流程。
### Lines 236-241: Namespace llvm
```cpp

namespace llvm {
FunctionPass *createAArch64O0PreLegalizerCombiner() {
  return new AArch64O0PreLegalizerCombinerLegacy();
}
} // end namespace llvm
```
**EN:** This block scopes the following declarations to the intended namespace boundary.  
**CN:** 该代码块将后续声明限定在目标命名空间边界内。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
- **EN:** Making generic IR operations legal on the target **CN:** 将通用 IR 操作转换为目标可接受形式
- **EN:** Canonical combines that simplify later codegen **CN:** 简化后续代码生成的规范化合并
## Dependencies / 依赖关系
- **EN:** Target-local includes: AArch64.h, AArch64GlobalISelUtils.h, AArch64TargetMachine.h, AArch64GenO0PreLegalizeGICombiner.inc, AArch64GenO0PreLegalizeGICombiner.inc, AArch64GenO0PreLegalizeGICombiner.inc, AArch64GenO0PreLegalizeGICombiner.inc, AArch64GenO0PreLegalizeGICombiner.inc **CN:** 目标本地依赖：AArch64.h, AArch64GlobalISelUtils.h, AArch64TargetMachine.h, AArch64GenO0PreLegalizeGICombiner.inc, AArch64GenO0PreLegalizeGICombiner.inc, AArch64GenO0PreLegalizeGICombiner.inc, AArch64GenO0PreLegalizeGICombiner.inc, AArch64GenO0PreLegalizeGICombiner.inc
- **EN:** Core LLVM interfaces: llvm/CodeGen/GlobalISel/Combiner.h, llvm/CodeGen/GlobalISel/CombinerHelper.h, llvm/CodeGen/GlobalISel/CombinerInfo.h, llvm/CodeGen/GlobalISel/GIMatchTableExecutorImpl.h, llvm/CodeGen/GlobalISel/GISelValueTracking.h, llvm/CodeGen/GlobalISel/MIPatternMatch.h, llvm/CodeGen/GlobalISel/MachineIRBuilder.h, llvm/CodeGen/LibcallLoweringInfo.h, llvm/CodeGen/MachineDominators.h, llvm/CodeGen/MachineFunction.h ... **CN:** 核心 LLVM 接口：llvm/CodeGen/GlobalISel/Combiner.h, llvm/CodeGen/GlobalISel/CombinerHelper.h, llvm/CodeGen/GlobalISel/CombinerInfo.h, llvm/CodeGen/GlobalISel/GIMatchTableExecutorImpl.h, llvm/CodeGen/GlobalISel/GISelValueTracking.h, llvm/CodeGen/GlobalISel/MIPatternMatch.h, llvm/CodeGen/GlobalISel/MachineIRBuilder.h, llvm/CodeGen/LibcallLoweringInfo.h, llvm/CodeGen/MachineDominators.h, llvm/CodeGen/MachineFunction.h ...
- **EN:** Standard-library support: memory **CN:** 标准库支持：memory
- **EN:** Closely connected with neighboring AArch64 backend components responsible for type and operation legalization. **CN:** 与周边负责类型与操作合法化的 AArch64 后端组件紧密协作。
