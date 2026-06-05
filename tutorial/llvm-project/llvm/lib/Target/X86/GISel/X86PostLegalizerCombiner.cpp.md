# X86PostLegalizerCombiner.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/GISel/X86PostLegalizerCombiner.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements legalization rules for X86 GlobalISel. / 实现X86 GlobalISel中的合法化规则。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//===--------------- X86PostLegalizerCombiner.cpp ---------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// Post-legalization combines on generic MachineInstrs.
///
/// The combines here must preserve instruction legality.
///
/// Lowering combines (e.g. pseudo matching) should be handled by
/// X86PostLegalizerLowering.
///
/// Combines which don't rely on instruction legality should go in the
/// X86PreLegalizerCombiner.
///
//===----------------------------------------------------------------------===//
#include "X86.h"
#include "X86TargetMachine.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/CodeGen/GlobalISel/CSEInfo.h"
#include "llvm/CodeGen/GlobalISel/CSEMIRBuilder.h"
#include "llvm/CodeGen/GlobalISel/Combiner.h"
#include "llvm/CodeGen/GlobalISel/CombinerHelper.h"
#include "llvm/CodeGen/GlobalISel/CombinerInfo.h"
#include "llvm/CodeGen/GlobalISel/GIMatchTableExecutorImpl.h"
#include "llvm/CodeGen/GlobalISel/GISelChangeObserver.h"
```
**EN:** This section implements file header, licensing, and opening context for X86 GlobalISel. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 GlobalISel中的文件头、许可证与开场上下文。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 31-60: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
#include "llvm/CodeGen/GlobalISel/GISelValueTracking.h"
#include "llvm/CodeGen/GlobalISel/GenericMachineInstrs.h"
#include "llvm/CodeGen/GlobalISel/MIPatternMatch.h"
#include "llvm/CodeGen/GlobalISel/MachineIRBuilder.h"
#include "llvm/CodeGen/GlobalISel/Utils.h"
#include "llvm/CodeGen/MachineDominators.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/TargetOpcodes.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/Support/Debug.h"

#define GET_GICOMBINER_DEPS
#include "X86GenPostLegalizeGICombiner.inc"
#undef GET_GICOMBINER_DEPS

#define DEBUG_TYPE "X86-postlegalizer-combiner"

using namespace llvm;
using namespace MIPatternMatch;

namespace {

CombinerInfo createCombinerInfo(bool OptEnabled, const Function &F) {
  CombinerInfo CInfo(/*AllowIllegalOps=*/true,
                     /*ShouldLegalizeIllegal=*/false,
                     /*LInfo=*/nullptr, /*OptEnabled=*/OptEnabled,
                     /*OptSize=*/F.hasOptSize(), /*MinSize=*/F.hasMinSize());
  // Disable fixed-point iteration to reduce compile-time
  CInfo.MaxIterations = 1;
```
**EN:** This section implements preprocessor directives and structural setup for X86 GlobalISel. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 GlobalISel中的预处理指令与结构初始化。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 61-90: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
  CInfo.ObserverLvl = CombinerInfo::ObserverLevel::SinglePass;
  // Legalizer performs DCE, so a full DCE pass is unnecessary.
  CInfo.EnableFullDCE = false;
  return CInfo;
}

#define GET_GICOMBINER_TYPES
#include "X86GenPostLegalizeGICombiner.inc"
#undef GET_GICOMBINER_TYPES

class X86PostLegalizerCombinerImpl : public Combiner {
protected:
  const CombinerHelper Helper;
  const X86PostLegalizerCombinerImplRuleConfig &RuleConfig;
  const X86Subtarget &STI;

public:
  X86PostLegalizerCombinerImpl(
      MachineFunction &MF, CombinerInfo &CInfo, GISelValueTracking &VT,
      GISelCSEInfo *CSEInfo,
      const X86PostLegalizerCombinerImplRuleConfig &RuleConfig,
      MachineDominatorTree *MDT);

  static const char *getName() { return "X86PostLegalizerCombiner"; }

  bool tryCombineAll(MachineInstr &I) const override;
  bool tryCombineAllImpl(MachineInstr &I) const;

private:
#define GET_GICOMBINER_CLASS_MEMBERS
```
**EN:** This section implements preprocessor directives and structural setup for X86 GlobalISel. Key symbols include X86PostLegalizerCombinerImpl. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 GlobalISel中的预处理指令与结构初始化。关键符号包括 X86PostLegalizerCombinerImpl。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 91-120: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
#include "X86GenPostLegalizeGICombiner.inc"
#undef GET_GICOMBINER_CLASS_MEMBERS
};

#define GET_GICOMBINER_IMPL
#include "X86GenPostLegalizeGICombiner.inc"
#undef GET_GICOMBINER_IMPL

X86PostLegalizerCombinerImpl::X86PostLegalizerCombinerImpl(
    MachineFunction &MF, CombinerInfo &CInfo, GISelValueTracking &VT,
    GISelCSEInfo *CSEInfo,
    const X86PostLegalizerCombinerImplRuleConfig &RuleConfig,
    MachineDominatorTree *MDT)
    : Combiner(MF, CInfo, &VT, CSEInfo),
      Helper(Observer, B, /*IsPreLegalize=*/false, &VT, MDT,
             MF.getSubtarget<X86Subtarget>().getLegalizerInfo()),
      RuleConfig(RuleConfig), STI(MF.getSubtarget<X86Subtarget>()),
#define GET_GICOMBINER_CONSTRUCTOR_INITS
#include "X86GenPostLegalizeGICombiner.inc"
#undef GET_GICOMBINER_CONSTRUCTOR_INITS
{
}

bool X86PostLegalizerCombinerImpl::tryCombineAll(MachineInstr &MI) const {
  return tryCombineAllImpl(MI);
}

class X86PostLegalizerCombinerLegacy : public MachineFunctionPass {
public:
  static char ID;
```
**EN:** This section implements preprocessor directives and structural setup for X86 GlobalISel. Key symbols include X86PostLegalizerCombinerLegacy. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 GlobalISel中的预处理指令与结构初始化。关键符号包括 X86PostLegalizerCombinerLegacy。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 121-150: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp

  X86PostLegalizerCombinerLegacy();

  StringRef getPassName() const override {
    return "X86PostLegalizerCombinerLegacy";
  }

  bool runOnMachineFunction(MachineFunction &MF) override;
  void getAnalysisUsage(AnalysisUsage &AU) const override;

private:
  X86PostLegalizerCombinerImplRuleConfig RuleConfig;
};
} // end anonymous namespace

void X86PostLegalizerCombinerLegacy::getAnalysisUsage(AnalysisUsage &AU) const {
  AU.addRequired<TargetPassConfig>();
  AU.setPreservesCFG();
  getSelectionDAGFallbackAnalysisUsage(AU);
  AU.addRequired<GISelValueTrackingAnalysisLegacy>();
  AU.addPreserved<GISelValueTrackingAnalysisLegacy>();
  // This is only added when processing level is not OptNone.
  AU.addRequired<MachineDominatorTreeWrapperPass>();
  AU.addPreserved<MachineDominatorTreeWrapperPass>();
  AU.addRequired<GISelCSEAnalysisWrapperPass>();
  AU.addPreserved<GISelCSEAnalysisWrapperPass>();

  MachineFunctionPass::getAnalysisUsage(AU);
}

```
**EN:** This section implements function implementation and target-specific behavior for X86 GlobalISel. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 GlobalISel中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 151-180: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
X86PostLegalizerCombinerLegacy::X86PostLegalizerCombinerLegacy()
    : MachineFunctionPass(ID) {
  if (!RuleConfig.parseCommandLineOption())
    reportFatalInternalError("Invalid rule identifier");
}

bool X86PostLegalizerCombinerLegacy::runOnMachineFunction(MachineFunction &MF) {
  if (MF.getProperties().hasFailedISel())
    return false;
  assert(MF.getProperties().hasLegalized() && "Expected a legalized function?");
  auto *TPC = &getAnalysis<TargetPassConfig>();
  const Function &F = MF.getFunction();

  GISelValueTracking *VT =
      &getAnalysis<GISelValueTrackingAnalysisLegacy>().get(MF);
  MachineDominatorTree *MDT =
      &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
  GISelCSEAnalysisWrapper &Wrapper =
      getAnalysis<GISelCSEAnalysisWrapperPass>().getCSEWrapper();
  auto *CSEInfo = &Wrapper.get(TPC->getCSEConfig());

  CombinerInfo CInfo = createCombinerInfo(!skipFunction(F), F);

  X86PostLegalizerCombinerImpl Impl(MF, CInfo, *VT, CSEInfo, RuleConfig, MDT);
  return Impl.combineMachineInstrs();
}

char X86PostLegalizerCombinerLegacy::ID = 0;
INITIALIZE_PASS_BEGIN(X86PostLegalizerCombinerLegacy, DEBUG_TYPE,
                      "Combine X86 MachineInstrs after legalization", false,
```
**EN:** This section implements function implementation and target-specific behavior for X86 GlobalISel. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 GlobalISel中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 181-210: Namespace scope management / 命名空间作用域管理
```cpp
                      false)
INITIALIZE_PASS_DEPENDENCY(TargetPassConfig)
INITIALIZE_PASS_DEPENDENCY(GISelValueTrackingAnalysisLegacy)
INITIALIZE_PASS_END(X86PostLegalizerCombinerLegacy, DEBUG_TYPE,
                    "Combine X86 MachineInstrs after legalization", false,
                    false)

namespace llvm {

PreservedAnalyses
X86PostLegalizerCombinerPass::run(MachineFunction &MF,
                                  MachineFunctionAnalysisManager &MFAM) {
  if (MF.getProperties().hasFailedISel())
    return PreservedAnalyses::all();
  assert(MF.getProperties().hasLegalized() && "Expected a legalized function.");
  const Function &F = MF.getFunction();

  GISelValueTracking VT = MFAM.getResult<GISelValueTrackingAnalysis>(MF);
  MachineDominatorTree &MDT = MFAM.getResult<MachineDominatorTreeAnalysis>(MF);
  auto &CSEInfo = MFAM.getResult<GISelCSEAnalysis>(MF);

  CombinerInfo CInfo = createCombinerInfo(true, F);

  X86PostLegalizerCombinerImplRuleConfig RuleConfig;
  if (!RuleConfig.parseCommandLineOption())
    reportFatalInternalError("Invalid rule identifier");

  X86PostLegalizerCombinerImpl Impl(MF, CInfo, VT, CSEInfo.get(), RuleConfig,
                                    &MDT);
  if (!Impl.combineMachineInstrs())
```
**EN:** This section implements namespace scope management for X86 GlobalISel. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 GlobalISel中的命名空间作用域管理。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 211-223: Supporting logic and local data handling / 辅助逻辑与局部数据处理
```cpp
    return PreservedAnalyses::all();

  PreservedAnalyses PA = getMachineFunctionPassPreservedAnalyses();
  PA.preserveSet<CFGAnalyses>();
  PA.preserve<GISelCSEAnalysis>();
  PA.preserve<GISelValueTrackingAnalysis>();
  return PA;
}

FunctionPass *createX86PostLegalizerCombinerLegacy() {
  return new X86PostLegalizerCombinerLegacy();
}
} // end namespace llvm
```
**EN:** This section implements supporting logic and local data handling for X86 GlobalISel. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 GlobalISel中的辅助逻辑与局部数据处理。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

## Key Concepts / 关键概念
- Primary topic: legalization rules. / 核心主题：合法化规则。
- Subsystem: X86 GlobalISel. / 所属子系统：X86 GlobalISel。
- Notable symbols: X86PostLegalizerCombinerImpl, X86PostLegalizerCombinerLegacy. / 重要符号：X86PostLegalizerCombinerImpl, X86PostLegalizerCombinerLegacy。
- The file emphasizes executable backend logic and target-specific decision making. / 该文件以可执行后端逻辑和目标特定决策为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: X86.h, X86TargetMachine.h, llvm/ADT/STLExtras.h, llvm/CodeGen/GlobalISel/CSEInfo.h, llvm/CodeGen/GlobalISel/CSEMIRBuilder.h, llvm/CodeGen/GlobalISel/Combiner.h, llvm/CodeGen/GlobalISel/CombinerHelper.h, llvm/CodeGen/GlobalISel/CombinerInfo.h, llvm/CodeGen/GlobalISel/GIMatchTableExecutorImpl.h, llvm/CodeGen/GlobalISel/GISelChangeObserver.h. / 直接包含：X86.h, X86TargetMachine.h, llvm/ADT/STLExtras.h, llvm/CodeGen/GlobalISel/CSEInfo.h, llvm/CodeGen/GlobalISel/CSEMIRBuilder.h, llvm/CodeGen/GlobalISel/Combiner.h, llvm/CodeGen/GlobalISel/CombinerHelper.h, llvm/CodeGen/GlobalISel/CombinerInfo.h, llvm/CodeGen/GlobalISel/GIMatchTableExecutorImpl.h, llvm/CodeGen/GlobalISel/GISelChangeObserver.h。
- Additional include dependencies: 16 more headers. / 额外包含依赖：还有 16 个头文件。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
