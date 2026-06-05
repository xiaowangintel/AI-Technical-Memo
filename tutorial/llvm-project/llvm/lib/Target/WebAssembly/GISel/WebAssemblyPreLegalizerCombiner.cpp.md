# WebAssemblyPreLegalizerCombiner.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/GISel/WebAssemblyPreLegalizerCombiner.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides GlobalISel legalization rules for the WebAssembly backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/GISel/WebAssemblyPreLegalizerCombiner.cpp`，主要负责 WebAssembly 后端的GlobalISel 合法化规则。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//=== WebAssemblyPreLegalizerCombiner.cpp ---------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-11

```cpp
//===----------------------------------------------------------------------===//
//
// This pass does combining of machine instructions at the generic MI level,
// before the legalizer.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This pass does combining of machine instructions at the generic MI level,".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This pass does combining of machine instructions at the generic MI level,”。

### Lines 12-18

```cpp
//===----------------------------------------------------------------------===//

#include "WebAssembly.h"
#include "WebAssemblySubtarget.h"
#include "llvm/CodeGen/GlobalISel/CSEInfo.h"
#include "llvm/CodeGen/GlobalISel/Combiner.h"
#include "llvm/CodeGen/GlobalISel/CombinerHelper.h"
```
- **EN**: Pulls in direct dependencies required by this GlobalISel legalization rules, so later declarations can reuse LLVM infrastructure and target-specific helpers. This range participates in the GlobalISel pipeline.
- **CN**: 这一段引入该GlobalISel 合法化规则所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段参与 GlobalISel 流水线。

### Lines 19-25

```cpp
#include "llvm/CodeGen/GlobalISel/CombinerInfo.h"
#include "llvm/CodeGen/GlobalISel/GIMatchTableExecutorImpl.h"
#include "llvm/CodeGen/GlobalISel/GISelValueTracking.h"
#include "llvm/CodeGen/GlobalISel/MachineIRBuilder.h"
#include "llvm/CodeGen/MachineDominators.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
```
- **EN**: Pulls in direct dependencies required by this GlobalISel legalization rules, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该GlobalISel 合法化规则所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 26-32

```cpp
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/Target/TargetMachine.h"

#define GET_GICOMBINER_DEPS
#include "WebAssemblyGenPreLegalizeGICombiner.inc"
#undef GET_GICOMBINER_DEPS
```
- **EN**: Pulls in direct dependencies required by this GlobalISel legalization rules, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该GlobalISel 合法化规则所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 33-39

```cpp
#define DEBUG_TYPE "wasm-prelegalizer-combiner"

using namespace llvm;

namespace {

#define GET_GICOMBINER_TYPES
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

### Lines 40-61

```cpp
#include "WebAssemblyGenPreLegalizeGICombiner.inc"
#undef GET_GICOMBINER_TYPES

class WebAssemblyPreLegalizerCombinerImpl : public Combiner {
protected:
  const CombinerHelper Helper;
  const WebAssemblyPreLegalizerCombinerImplRuleConfig &RuleConfig;
  const WebAssemblySubtarget &STI;

public:
  WebAssemblyPreLegalizerCombinerImpl(
      MachineFunction &MF, CombinerInfo &CInfo, GISelValueTracking &VT,
      GISelCSEInfo *CSEInfo,
      const WebAssemblyPreLegalizerCombinerImplRuleConfig &RuleConfig,
      const WebAssemblySubtarget &STI, MachineDominatorTree *MDT,
      const LegalizerInfo *LI);

  static const char *getName() { return "WebAssembly00PreLegalizerCombiner"; }

  bool tryCombineAll(MachineInstr &I) const override;

private:
```
- **EN**: Pulls in direct dependencies required by this GlobalISel legalization rules, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该GlobalISel 合法化规则所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 62-79

```cpp
#define GET_GICOMBINER_CLASS_MEMBERS
#include "WebAssemblyGenPreLegalizeGICombiner.inc"
#undef GET_GICOMBINER_CLASS_MEMBERS
};

#define GET_GICOMBINER_IMPL
#include "WebAssemblyGenPreLegalizeGICombiner.inc"
#undef GET_GICOMBINER_IMPL

WebAssemblyPreLegalizerCombinerImpl::WebAssemblyPreLegalizerCombinerImpl(
    MachineFunction &MF, CombinerInfo &CInfo, GISelValueTracking &VT,
    GISelCSEInfo *CSEInfo,
    const WebAssemblyPreLegalizerCombinerImplRuleConfig &RuleConfig,
    const WebAssemblySubtarget &STI, MachineDominatorTree *MDT,
    const LegalizerInfo *LI)
    : Combiner(MF, CInfo, &VT, CSEInfo),
      Helper(Observer, B, /*IsPreLegalize*/ true, &VT, MDT, LI),
      RuleConfig(RuleConfig), STI(STI),
```
- **EN**: Pulls in direct dependencies required by this GlobalISel legalization rules, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该GlobalISel 合法化规则所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 80-88

```cpp
#define GET_GICOMBINER_CONSTRUCTOR_INITS
#include "WebAssemblyGenPreLegalizeGICombiner.inc"
#undef GET_GICOMBINER_CONSTRUCTOR_INITS
{
}

// Pass boilerplate
// ================
```
- **EN**: Pulls in direct dependencies required by this GlobalISel legalization rules, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该GlobalISel 合法化规则所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 89-126

```cpp
class WebAssemblyPreLegalizerCombiner : public MachineFunctionPass {
public:
  static char ID;

  WebAssemblyPreLegalizerCombiner();

  StringRef getPassName() const override {
    return "WebAssemblyPreLegalizerCombiner";
  }

  bool runOnMachineFunction(MachineFunction &MF) override;

  void getAnalysisUsage(AnalysisUsage &AU) const override;

private:
  WebAssemblyPreLegalizerCombinerImplRuleConfig RuleConfig;
};
} // end anonymous namespace

void WebAssemblyPreLegalizerCombiner::getAnalysisUsage(
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

WebAssemblyPreLegalizerCombiner::WebAssemblyPreLegalizerCombiner()
    : MachineFunctionPass(ID) {
  if (!RuleConfig.parseCommandLineOption())
    report_fatal_error("Invalid rule identifier");
}
```
- **EN**: Declares a backend-facing type `WebAssemblyPreLegalizerCombiner`, `getPassName`, `runOnMachineFunction` and outlines the API or state that nearby code will rely on. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里声明面向后端的类型 `WebAssemblyPreLegalizerCombiner`, `getPassName`, `runOnMachineFunction`，并勾勒出周边代码会依赖的接口或状态。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 127-164

```cpp

bool WebAssemblyPreLegalizerCombiner::runOnMachineFunction(
    MachineFunction &MF) {
  if (MF.getProperties().hasFailedISel())
    return false;
  auto &TPC = getAnalysis<TargetPassConfig>();

  // Enable CSE.
  GISelCSEAnalysisWrapper &Wrapper =
      getAnalysis<GISelCSEAnalysisWrapperPass>().getCSEWrapper();
  auto *CSEInfo = &Wrapper.get(TPC.getCSEConfig());

  const WebAssemblySubtarget &ST = MF.getSubtarget<WebAssemblySubtarget>();
  const auto *LI = ST.getLegalizerInfo();

  const Function &F = MF.getFunction();
  bool EnableOpt =
      MF.getTarget().getOptLevel() != CodeGenOptLevel::None && !skipFunction(F);
  GISelValueTracking *VT =
      &getAnalysis<GISelValueTrackingAnalysisLegacy>().get(MF);
  MachineDominatorTree *MDT =
      &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
  CombinerInfo CInfo(/*AllowIllegalOps*/ true, /*ShouldLegalizeIllegal*/ false,
                     /*LegalizerInfo*/ nullptr, EnableOpt, F.hasOptSize(),
                     F.hasMinSize());
  // Disable fixed-point iteration to reduce compile-time
  CInfo.MaxIterations = 1;
  CInfo.ObserverLvl = CombinerInfo::ObserverLevel::SinglePass;
  // This is the first Combiner, so the input IR might contain dead
  // instructions.
  CInfo.EnableFullDCE = true;
  WebAssemblyPreLegalizerCombinerImpl Impl(MF, CInfo, *VT, CSEInfo, RuleConfig,
                                           ST, MDT, LI);
  return Impl.combineMachineInstrs();
}

char WebAssemblyPreLegalizerCombiner::ID = 0;
INITIALIZE_PASS_BEGIN(WebAssemblyPreLegalizerCombiner, DEBUG_TYPE,
```
- **EN**: Implements helper routine(s) `runOnMachineFunction`, `getProperties`, `hasFailedISel` for this portion of the WebAssembly backend GlobalISel legalization rules. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分GlobalISel 合法化规则所需的辅助例程 `runOnMachineFunction`, `getProperties`, `hasFailedISel`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 165-176

```cpp
                      "Combine WebAssembly machine instrs before legalization",
                      false, false)
INITIALIZE_PASS_DEPENDENCY(TargetPassConfig)
INITIALIZE_PASS_DEPENDENCY(GISelValueTrackingAnalysisLegacy)
INITIALIZE_PASS_DEPENDENCY(GISelCSEAnalysisWrapperPass)
INITIALIZE_PASS_END(WebAssemblyPreLegalizerCombiner, DEBUG_TYPE,
                    "Combine WebAssembly machine instrs before legalization",
                    false, false)

FunctionPass *llvm::createWebAssemblyPreLegalizerCombiner() {
  return new WebAssemblyPreLegalizerCombiner();
}
```
- **EN**: Implements helper routine(s) `INITIALIZE_PASS_DEPENDENCY`, `INITIALIZE_PASS_END`, `createWebAssemblyPreLegalizerCombiner` for this portion of the WebAssembly backend GlobalISel legalization rules.
- **CN**: 这里实现了 WebAssembly 后端该部分GlobalISel 合法化规则所需的辅助例程 `INITIALIZE_PASS_DEPENDENCY`, `INITIALIZE_PASS_END`, `createWebAssemblyPreLegalizerCombiner`。

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
- `WebAssemblySubtarget.h`
- `llvm/CodeGen/GlobalISel/CSEInfo.h`
- `llvm/CodeGen/GlobalISel/Combiner.h`
- `llvm/CodeGen/GlobalISel/CombinerHelper.h`
- `llvm/CodeGen/GlobalISel/CombinerInfo.h`
- `llvm/CodeGen/GlobalISel/GIMatchTableExecutorImpl.h`
- `llvm/CodeGen/GlobalISel/GISelValueTracking.h`
- `llvm/CodeGen/GlobalISel/MachineIRBuilder.h`
- `llvm/CodeGen/MachineDominators.h`
- `llvm/CodeGen/MachineFunction.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/TargetPassConfig.h`
- `llvm/Target/TargetMachine.h`
- `WebAssemblyGenPreLegalizeGICombiner.inc`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
