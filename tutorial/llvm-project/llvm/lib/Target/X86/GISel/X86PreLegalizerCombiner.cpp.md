# X86PreLegalizerCombiner.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/GISel/X86PreLegalizerCombiner.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements legalization rules for X86 GlobalISel. / 实现X86 GlobalISel中的合法化规则。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//===---------------- X86PreLegalizerCombiner.cpp -------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
/// \file
/// This pass does combining of machine instructions at the generic MI level,
/// before the legalizer.
///
//===----------------------------------------------------------------------===//
#include "X86.h"
#include "X86TargetMachine.h"
#include "llvm/CodeGen/GlobalISel/CSEInfo.h"
#include "llvm/CodeGen/GlobalISel/Combiner.h"
#include "llvm/CodeGen/GlobalISel/CombinerHelper.h"
#include "llvm/CodeGen/GlobalISel/CombinerInfo.h"
#include "llvm/CodeGen/GlobalISel/GIMatchTableExecutorImpl.h"
#include "llvm/CodeGen/GlobalISel/GISelValueTracking.h"
#include "llvm/CodeGen/GlobalISel/LegalizerInfo.h"
#include "llvm/CodeGen/GlobalISel/MIPatternMatch.h"
#include "llvm/CodeGen/GlobalISel/MachineIRBuilder.h"
#include "llvm/CodeGen/GlobalISel/Utils.h"
#include "llvm/CodeGen/MachineDominators.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/IR/Instructions.h"
```
**EN:** This section implements file header, licensing, and opening context for X86 GlobalISel. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 GlobalISel中的文件头、许可证与开场上下文。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 31-60: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp

#define GET_GICOMBINER_DEPS
#include "X86GenPreLegalizeGICombiner.inc"
#undef GET_GICOMBINER_DEPS

#define DEBUG_TYPE "x86-prelegalizer-combiner"

using namespace llvm;
using namespace MIPatternMatch;

namespace {

CombinerInfo createCombinerInfo(bool EnableOpt, const Function &F) {
  CombinerInfo CInfo(/*AllowIllegalOps=*/true, /*ShouldLegalizeIllegal=*/false,
                     nullptr, EnableOpt, F.hasOptSize(), F.hasMinSize());

  // This is the first Combiner, so the input IR might contain dead
  // instructions.
  CInfo.EnableFullDCE = true;
  return CInfo;
}

#define GET_GICOMBINER_TYPES
#include "X86GenPreLegalizeGICombiner.inc"
#undef GET_GICOMBINER_TYPES

class X86PreLegalizerCombinerImpl : public Combiner {
protected:
  const CombinerHelper Helper;
  const X86PreLegalizerCombinerImplRuleConfig &RuleConfig;
```
**EN:** This section implements preprocessor directives and structural setup for X86 GlobalISel. Key symbols include X86PreLegalizerCombinerImpl. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 GlobalISel中的预处理指令与结构初始化。关键符号包括 X86PreLegalizerCombinerImpl。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 61-90: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
  const X86Subtarget &STI;

public:
  X86PreLegalizerCombinerImpl(
      MachineFunction &MF, CombinerInfo &CInfo, GISelValueTracking &VT,
      GISelCSEInfo *CSEInfo,
      const X86PreLegalizerCombinerImplRuleConfig &RuleConfig,
      MachineDominatorTree *MDT);

  static const char *getName() { return "X86PreLegalizerCombiner"; }

  bool tryCombineAll(MachineInstr &I) const override;

  bool tryCombineAllImpl(MachineInstr &I) const;

private:
#define GET_GICOMBINER_CLASS_MEMBERS
#include "X86GenPreLegalizeGICombiner.inc"
#undef GET_GICOMBINER_CLASS_MEMBERS
};

#define GET_GICOMBINER_IMPL
#include "X86GenPreLegalizeGICombiner.inc"
#undef GET_GICOMBINER_IMPL

X86PreLegalizerCombinerImpl::X86PreLegalizerCombinerImpl(
    MachineFunction &MF, CombinerInfo &CInfo, GISelValueTracking &VT,
    GISelCSEInfo *CSEInfo,
    const X86PreLegalizerCombinerImplRuleConfig &RuleConfig,
    MachineDominatorTree *MDT)
```
**EN:** This section implements preprocessor directives and structural setup for X86 GlobalISel. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 GlobalISel中的预处理指令与结构初始化。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 91-120: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
    : Combiner(MF, CInfo, &VT, CSEInfo),
      Helper(Observer, B, /*IsPreLegalize=*/true, &VT, MDT,
             MF.getSubtarget<X86Subtarget>().getLegalizerInfo()),
      RuleConfig(RuleConfig), STI(MF.getSubtarget<X86Subtarget>()),
#define GET_GICOMBINER_CONSTRUCTOR_INITS
#include "X86GenPreLegalizeGICombiner.inc"
#undef GET_GICOMBINER_CONSTRUCTOR_INITS
{
}

bool X86PreLegalizerCombinerImpl::tryCombineAll(MachineInstr &MI) const {
  return tryCombineAllImpl(MI);
}

class X86PreLegalizerCombinerLegacy : public MachineFunctionPass {
public:
  static char ID;

  X86PreLegalizerCombinerLegacy();

  StringRef getPassName() const override {
    return "X86PreLegalizerCombinerLegacy";
  }

  bool runOnMachineFunction(MachineFunction &MF) override;

  void getAnalysisUsage(AnalysisUsage &AU) const override;

private:
  X86PreLegalizerCombinerImplRuleConfig RuleConfig;
```
**EN:** This section implements preprocessor directives and structural setup for X86 GlobalISel. Key symbols include X86PreLegalizerCombinerLegacy, Combiner. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 GlobalISel中的预处理指令与结构初始化。关键符号包括 X86PreLegalizerCombinerLegacy, Combiner。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 121-150: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
};
} // end anonymous namespace

void X86PreLegalizerCombinerLegacy::getAnalysisUsage(AnalysisUsage &AU) const {
  AU.addRequired<TargetPassConfig>();
  AU.setPreservesCFG();
  getSelectionDAGFallbackAnalysisUsage(AU);
  AU.addRequired<GISelValueTrackingAnalysisLegacy>();
  AU.addPreserved<GISelValueTrackingAnalysisLegacy>();
  AU.addRequired<MachineDominatorTreeWrapperPass>();
  AU.addPreserved<MachineDominatorTreeWrapperPass>();
  AU.addRequired<GISelCSEAnalysisWrapperPass>();
  AU.addPreserved<GISelCSEAnalysisWrapperPass>();
  MachineFunctionPass::getAnalysisUsage(AU);
}

X86PreLegalizerCombinerLegacy::X86PreLegalizerCombinerLegacy()
    : MachineFunctionPass(ID) {
  if (!RuleConfig.parseCommandLineOption())
    report_fatal_error("Invalid rule identifier");
}

bool X86PreLegalizerCombinerLegacy::runOnMachineFunction(MachineFunction &MF) {
  if (MF.getProperties().hasFailedISel())
    return false;
  auto &TPC = getAnalysis<TargetPassConfig>();

  // Enable CSE.
  GISelCSEAnalysisWrapper &Wrapper =
      getAnalysis<GISelCSEAnalysisWrapperPass>().getCSEWrapper();
```
**EN:** This section implements function implementation and target-specific behavior for X86 GlobalISel. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 GlobalISel中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 151-180: Namespace scope management / 命名空间作用域管理
```cpp
  auto *CSEInfo = &Wrapper.get(TPC.getCSEConfig());
  const Function &F = MF.getFunction();
  bool EnableOpt =
      MF.getTarget().getOptLevel() != CodeGenOptLevel::None && !skipFunction(F);
  GISelValueTracking *VT =
      &getAnalysis<GISelValueTrackingAnalysisLegacy>().get(MF);
  MachineDominatorTree *MDT =
      &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
  CombinerInfo CInfo = createCombinerInfo(EnableOpt, F);
  X86PreLegalizerCombinerImpl Impl(MF, CInfo, *VT, CSEInfo, RuleConfig, MDT);
  return Impl.combineMachineInstrs();
}

char X86PreLegalizerCombinerLegacy::ID = 0;
INITIALIZE_PASS_BEGIN(X86PreLegalizerCombinerLegacy, DEBUG_TYPE,
                      "Combine X86 machine instrs before legalization", false,
                      false)
INITIALIZE_PASS_DEPENDENCY(TargetPassConfig)
INITIALIZE_PASS_DEPENDENCY(GISelValueTrackingAnalysisLegacy)
INITIALIZE_PASS_DEPENDENCY(GISelCSEAnalysisWrapperPass)
INITIALIZE_PASS_END(X86PreLegalizerCombinerLegacy, DEBUG_TYPE,
                    "Combine X86 machine instrs before legalization", false,
                    false)

namespace llvm {

PreservedAnalyses
X86PreLegalizerCombinerPass::run(MachineFunction &MF,
                                 MachineFunctionAnalysisManager &MFAM) {
  if (MF.getProperties().hasFailedISel())
```
**EN:** This section implements namespace scope management for X86 GlobalISel. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 GlobalISel中的命名空间作用域管理。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 181-207: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp
    return PreservedAnalyses::all();

  X86PreLegalizerCombinerImplRuleConfig RuleConfig;
  if (!RuleConfig.parseCommandLineOption())
    report_fatal_error("Invalid rule identifier");

  auto &CSEInfo = MFAM.getResult<GISelCSEAnalysis>(MF);
  const Function &F = MF.getFunction();
  bool EnableOpt = MF.getTarget().getOptLevel() != CodeGenOptLevel::None;
  GISelValueTracking &VT = MFAM.getResult<GISelValueTrackingAnalysis>(MF);
  MachineDominatorTree &MDT = MFAM.getResult<MachineDominatorTreeAnalysis>(MF);
  CombinerInfo CInfo = createCombinerInfo(EnableOpt, F);
  X86PreLegalizerCombinerImpl Impl(MF, CInfo, VT, CSEInfo.get(), RuleConfig,
                                   &MDT);
  Impl.combineMachineInstrs();

  PreservedAnalyses PA = getMachineFunctionPassPreservedAnalyses();
  PA.preserveSet<CFGAnalyses>();
  PA.preserve<GISelCSEAnalysis>();
  PA.preserve<GISelValueTrackingAnalysis>();
  return PA;
}

FunctionPass *createX86PreLegalizerCombinerLegacy() {
  return new X86PreLegalizerCombinerLegacy();
}
} // end namespace llvm
```
**EN:** This section implements control flow, matching, and data updates for X86 GlobalISel. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 GlobalISel中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

## Key Concepts / 关键概念
- Primary topic: legalization rules. / 核心主题：合法化规则。
- Subsystem: X86 GlobalISel. / 所属子系统：X86 GlobalISel。
- Notable symbols: X86PreLegalizerCombinerImpl, X86PreLegalizerCombinerLegacy. / 重要符号：X86PreLegalizerCombinerImpl, X86PreLegalizerCombinerLegacy。
- The file emphasizes executable backend logic and target-specific decision making. / 该文件以可执行后端逻辑和目标特定决策为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: X86.h, X86TargetMachine.h, llvm/CodeGen/GlobalISel/CSEInfo.h, llvm/CodeGen/GlobalISel/Combiner.h, llvm/CodeGen/GlobalISel/CombinerHelper.h, llvm/CodeGen/GlobalISel/CombinerInfo.h, llvm/CodeGen/GlobalISel/GIMatchTableExecutorImpl.h, llvm/CodeGen/GlobalISel/GISelValueTracking.h, llvm/CodeGen/GlobalISel/LegalizerInfo.h, llvm/CodeGen/GlobalISel/MIPatternMatch.h. / 直接包含：X86.h, X86TargetMachine.h, llvm/CodeGen/GlobalISel/CSEInfo.h, llvm/CodeGen/GlobalISel/Combiner.h, llvm/CodeGen/GlobalISel/CombinerHelper.h, llvm/CodeGen/GlobalISel/CombinerInfo.h, llvm/CodeGen/GlobalISel/GIMatchTableExecutorImpl.h, llvm/CodeGen/GlobalISel/GISelValueTracking.h, llvm/CodeGen/GlobalISel/LegalizerInfo.h, llvm/CodeGen/GlobalISel/MIPatternMatch.h。
- Additional include dependencies: 13 more headers. / 额外包含依赖：还有 13 个头文件。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
