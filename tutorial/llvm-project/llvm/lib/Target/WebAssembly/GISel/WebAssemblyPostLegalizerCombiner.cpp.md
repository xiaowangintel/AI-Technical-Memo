# WebAssemblyPostLegalizerCombiner.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/GISel/WebAssemblyPostLegalizerCombiner.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Post-legalization combines on generic MachineInstrs.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/GISel/WebAssemblyPostLegalizerCombiner.cpp`，主要负责 WebAssembly 后端的GlobalISel 合法化规则。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//=== WebAssemblyPostLegalizerCombiner.cpp ----------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-16

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// Post-legalization combines on generic MachineInstrs.
///
/// The combines here must preserve instruction legality.
///
/// Combines which don't rely on instruction legality should go in the
/// WebAssemblyPreLegalizerCombiner.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 17-23

```cpp
//===----------------------------------------------------------------------===//

#include "WebAssembly.h"
#include "WebAssemblyTargetMachine.h"
#include "llvm/CodeGen/GlobalISel/CSEInfo.h"
#include "llvm/CodeGen/GlobalISel/Combiner.h"
#include "llvm/CodeGen/GlobalISel/CombinerHelper.h"
```
- **EN**: Pulls in direct dependencies required by this GlobalISel legalization rules, so later declarations can reuse LLVM infrastructure and target-specific helpers. This range participates in the GlobalISel pipeline.
- **CN**: 这一段引入该GlobalISel 合法化规则所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段参与 GlobalISel 流水线。

### Lines 24-31

```cpp
#include "llvm/CodeGen/GlobalISel/CombinerInfo.h"
#include "llvm/CodeGen/GlobalISel/GIMatchTableExecutorImpl.h"
#include "llvm/CodeGen/GlobalISel/GISelValueTracking.h"
#include "llvm/CodeGen/GlobalISel/MachineIRBuilder.h"
#include "llvm/CodeGen/MachineDominators.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/TargetPassConfig.h"
```
- **EN**: Pulls in direct dependencies required by this GlobalISel legalization rules, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该GlobalISel 合法化规则所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 32-39

```cpp
#define GET_GICOMBINER_DEPS
#include "WebAssemblyGenPostLegalizeGICombiner.inc"
#undef GET_GICOMBINER_DEPS

#define DEBUG_TYPE "wasm-postlegalizer-combiner"

using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this GlobalISel legalization rules, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该GlobalISel 合法化规则所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 40-64

```cpp
namespace {

#define GET_GICOMBINER_TYPES
#include "WebAssemblyGenPostLegalizeGICombiner.inc"
#undef GET_GICOMBINER_TYPES

class WebAssemblyPostLegalizerCombinerImpl : public Combiner {
protected:
  const CombinerHelper Helper;
  const WebAssemblyPostLegalizerCombinerImplRuleConfig &RuleConfig;
  const WebAssemblySubtarget &STI;

public:
  WebAssemblyPostLegalizerCombinerImpl(
      MachineFunction &MF, CombinerInfo &CInfo, GISelValueTracking &VT,
      GISelCSEInfo *CSEInfo,
      const WebAssemblyPostLegalizerCombinerImplRuleConfig &RuleConfig,
      const WebAssemblySubtarget &STI, MachineDominatorTree *MDT,
      const LegalizerInfo *LI);

  static const char *getName() { return "WebAssemblyPostLegalizerCombiner"; }

  bool tryCombineAll(MachineInstr &I) const override;

private:
```
- **EN**: Pulls in direct dependencies required by this GlobalISel legalization rules, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该GlobalISel 合法化规则所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 65-82

```cpp
#define GET_GICOMBINER_CLASS_MEMBERS
#include "WebAssemblyGenPostLegalizeGICombiner.inc"
#undef GET_GICOMBINER_CLASS_MEMBERS
};

#define GET_GICOMBINER_IMPL
#include "WebAssemblyGenPostLegalizeGICombiner.inc"
#undef GET_GICOMBINER_IMPL

WebAssemblyPostLegalizerCombinerImpl::WebAssemblyPostLegalizerCombinerImpl(
    MachineFunction &MF, CombinerInfo &CInfo, GISelValueTracking &VT,
    GISelCSEInfo *CSEInfo,
    const WebAssemblyPostLegalizerCombinerImplRuleConfig &RuleConfig,
    const WebAssemblySubtarget &STI, MachineDominatorTree *MDT,
    const LegalizerInfo *LI)
    : Combiner(MF, CInfo, &VT, CSEInfo),
      Helper(Observer, B, /*IsPreLegalize*/ false, &VT, MDT, LI),
      RuleConfig(RuleConfig), STI(STI),
```
- **EN**: Pulls in direct dependencies required by this GlobalISel legalization rules, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该GlobalISel 合法化规则所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 83-120

```cpp
#define GET_GICOMBINER_CONSTRUCTOR_INITS
#include "WebAssemblyGenPostLegalizeGICombiner.inc"
#undef GET_GICOMBINER_CONSTRUCTOR_INITS
{
}

class WebAssemblyPostLegalizerCombiner : public MachineFunctionPass {
public:
  static char ID;

  WebAssemblyPostLegalizerCombiner();

  StringRef getPassName() const override {
    return "WebAssemblyPostLegalizerCombiner";
  }

  bool runOnMachineFunction(MachineFunction &MF) override;
  void getAnalysisUsage(AnalysisUsage &AU) const override;

private:
  WebAssemblyPostLegalizerCombinerImplRuleConfig RuleConfig;
};
} // end anonymous namespace

void WebAssemblyPostLegalizerCombiner::getAnalysisUsage(
    AnalysisUsage &AU) const {
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
```
- **EN**: Pulls in direct dependencies required by this GlobalISel legalization rules, so later declarations can reuse LLVM infrastructure and target-specific helpers. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这一段引入该GlobalISel 合法化规则所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 121-158

```cpp
WebAssemblyPostLegalizerCombiner::WebAssemblyPostLegalizerCombiner()
    : MachineFunctionPass(ID) {
  if (!RuleConfig.parseCommandLineOption())
    report_fatal_error("Invalid rule identifier");
}

bool WebAssemblyPostLegalizerCombiner::runOnMachineFunction(
    MachineFunction &MF) {
  if (MF.getProperties().hasFailedISel())
    return false;
  assert(MF.getProperties().hasLegalized() && "Expected a legalized function?");
  auto *TPC = &getAnalysis<TargetPassConfig>();
  const Function &F = MF.getFunction();
  bool EnableOpt =
      MF.getTarget().getOptLevel() != CodeGenOptLevel::None && !skipFunction(F);

  const WebAssemblySubtarget &ST = MF.getSubtarget<WebAssemblySubtarget>();
  const auto *LI = ST.getLegalizerInfo();

  GISelValueTracking *VT =
      &getAnalysis<GISelValueTrackingAnalysisLegacy>().get(MF);
  MachineDominatorTree *MDT = nullptr;
  GISelCSEInfo *CSEInfo = nullptr;

  MDT = &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();

  GISelCSEAnalysisWrapper &Wrapper =
      getAnalysis<GISelCSEAnalysisWrapperPass>().getCSEWrapper();
  CSEInfo = &Wrapper.get(TPC->getCSEConfig());

  CombinerInfo CInfo(/*AllowIllegalOps*/ true, /*ShouldLegalizeIllegal*/ false,
                     /*LegalizerInfo*/ nullptr, EnableOpt, F.hasOptSize(),
                     F.hasMinSize());
  // Disable fixed-point iteration to reduce compile-time
  CInfo.MaxIterations = 1;
  CInfo.ObserverLvl = CombinerInfo::ObserverLevel::SinglePass;
  // Legalizer performs DCE, so a full DCE pass is unnecessary.
  CInfo.EnableFullDCE = false;
```
- **EN**: Implements helper routine(s) `WebAssemblyPostLegalizerCombiner`, `MachineFunctionPass`, `parseCommandLineOption` for this portion of the WebAssembly backend GlobalISel legalization rules. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分GlobalISel 合法化规则所需的辅助例程 `WebAssemblyPostLegalizerCombiner`, `MachineFunctionPass`, `parseCommandLineOption`。 子目标特性裁剪会影响这里的行为。

### Lines 159-176

```cpp
  WebAssemblyPostLegalizerCombinerImpl Impl(MF, CInfo, *VT, CSEInfo, RuleConfig,
                                            ST, MDT, LI);
  return Impl.combineMachineInstrs();
}

char WebAssemblyPostLegalizerCombiner::ID = 0;
INITIALIZE_PASS_BEGIN(WebAssemblyPostLegalizerCombiner, DEBUG_TYPE,
                      "Combine WebAssembly MachineInstrs after legalization",
                      false, false)
INITIALIZE_PASS_DEPENDENCY(TargetPassConfig)
INITIALIZE_PASS_DEPENDENCY(GISelValueTrackingAnalysisLegacy)
INITIALIZE_PASS_END(WebAssemblyPostLegalizerCombiner, DEBUG_TYPE,
                    "Combine WebAssembly MachineInstrs after legalization",
                    false, false)

FunctionPass *llvm::createWebAssemblyPostLegalizerCombiner() {
  return new WebAssemblyPostLegalizerCombiner();
}
```
- **EN**: Implements helper routine(s) `Impl`, `combineMachineInstrs`, `INITIALIZE_PASS_BEGIN` for this portion of the WebAssembly backend GlobalISel legalization rules. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分GlobalISel 合法化规则所需的辅助例程 `Impl`, `combineMachineInstrs`, `INITIALIZE_PASS_BEGIN`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- GlobalISel legalization rules / GlobalISel 合法化规则
- GlobalISel pipeline / GlobalISel 流水线
- SelectionDAG lowering / SelectionDAG lowering
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Type or operation legalization / 类型或操作合法化
- Target machine configuration / 目标机器配置

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssembly.h`
- `WebAssemblyTargetMachine.h`
- `llvm/CodeGen/GlobalISel/CSEInfo.h`
- `llvm/CodeGen/GlobalISel/Combiner.h`
- `llvm/CodeGen/GlobalISel/CombinerHelper.h`
- `llvm/CodeGen/GlobalISel/CombinerInfo.h`
- `llvm/CodeGen/GlobalISel/GIMatchTableExecutorImpl.h`
- `llvm/CodeGen/GlobalISel/GISelValueTracking.h`
- `llvm/CodeGen/GlobalISel/MachineIRBuilder.h`
- `llvm/CodeGen/MachineDominators.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/TargetPassConfig.h`
- `WebAssemblyGenPostLegalizeGICombiner.inc`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
