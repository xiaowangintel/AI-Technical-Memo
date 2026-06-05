# RISCVPostLegalizerCombiner.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/GISel/RISCVPostLegalizerCombiner.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements a post-legalization combiner for RISC-V GlobalISel machine IR. / 实现RISC-V GlobalISel Machine IR 的后合法化合并器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Commentary and design intent / 注释与设计意图
```cpp
//=== RISCVPostLegalizerCombiner.cpp --------------------------*- C++ -*-===//
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
/// Combines which don't rely on instruction legality should go in the
/// RISCVPreLegalizerCombiner.
///
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 18-31: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#include "RISCVTargetMachine.h"
#include "llvm/CodeGen/GlobalISel/CSEInfo.h"
#include "llvm/CodeGen/GlobalISel/Combiner.h"
#include "llvm/CodeGen/GlobalISel/CombinerHelper.h"
#include "llvm/CodeGen/GlobalISel/CombinerInfo.h"
#include "llvm/CodeGen/GlobalISel/GIMatchTableExecutorImpl.h"
#include "llvm/CodeGen/GlobalISel/GISelValueTracking.h"
#include "llvm/CodeGen/GlobalISel/MachineIRBuilder.h"
#include "llvm/CodeGen/MachineDominators.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/Support/FormatVariadic.h"
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 32-41: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp
#define GET_GICOMBINER_DEPS
#include "RISCVGenPostLegalizeGICombiner.inc"
#undef GET_GICOMBINER_DEPS

#define DEBUG_TYPE "riscv-postlegalizer-combiner"

using namespace llvm;

namespace {
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 42-52: File prologue and imported dependencies / 文件前言与导入依赖
```cpp
#define GET_GICOMBINER_TYPES
#include "RISCVGenPostLegalizeGICombiner.inc"
#undef GET_GICOMBINER_TYPES

/// Match: G_STORE (G_FCONSTANT +0.0), addr
/// Return the source vreg in MatchInfo if matched.
bool matchFoldFPZeroStore(MachineInstr &MI, MachineRegisterInfo &MRI,
                          const RISCVSubtarget &STI, Register &MatchInfo) {
  if (MI.getOpcode() != TargetOpcode::G_STORE)
    return false;
```
**EN:** This opening block combines banner comments with the first wave of includes, giving readers both context and the main compile-time dependencies.

**CN:** 该开头区段同时包含说明性注释与首批 include，让读者先看到文件定位，再看到主要编译期依赖。

### Lines 53-64: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  Register SrcReg = MI.getOperand(0).getReg();
  if (!SrcReg.isVirtual())
    return false;

  MachineInstr *Def = MRI.getVRegDef(SrcReg);
  if (!Def || Def->getOpcode() != TargetOpcode::G_FCONSTANT)
    return false;

  auto *CFP = Def->getOperand(1).getFPImm();
  if (!CFP || !CFP->getValueAPF().isPosZero())
    return false;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 65-74: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  unsigned ValBits = MRI.getType(SrcReg).getSizeInBits();
  if ((ValBits == 16 && !STI.hasStdExtZfh()) ||
      (ValBits == 32 && !STI.hasStdExtF()) ||
      (ValBits == 64 && (!STI.hasStdExtD() || !STI.is64Bit())))
    return false;

  MatchInfo = SrcReg;
  return true;
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 75-87: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
/// Apply: rewrite to G_STORE (G_CONSTANT 0 [XLEN]), addr
void applyFoldFPZeroStore(MachineInstr &MI, MachineRegisterInfo &MRI,
                          MachineIRBuilder &B, const RISCVSubtarget &STI,
                          Register &MatchInfo) {
  const unsigned XLen = STI.getXLen();

  auto Zero = B.buildConstant(LLT::scalar(XLen), 0);
  MI.getOperand(0).setReg(Zero.getReg(0));

  MachineInstr *Def = MRI.getVRegDef(MatchInfo);
  if (Def && MRI.use_nodbg_empty(MatchInfo))
    Def->eraseFromParent();
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 88-101: Header guard and interface framing / 头文件保护与接口框架
```cpp
#ifndef NDEBUG
  unsigned ValBits = MRI.getType(MatchInfo).getSizeInBits();
  LLVM_DEBUG(dbgs() << formatv("[{0}] Fold FP zero store -> int zero "
                               "(XLEN={1}, ValBits={2}):\n  {3}\n",
                               DEBUG_TYPE, XLen, ValBits, MI));
#endif
}

class RISCVPostLegalizerCombinerImpl : public Combiner {
protected:
  const CombinerHelper Helper;
  const RISCVPostLegalizerCombinerImplRuleConfig &RuleConfig;
  const RISCVSubtarget &STI;
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 102-111: Function implementation: getName / 函数实现：getName
```cpp
public:
  RISCVPostLegalizerCombinerImpl(
      MachineFunction &MF, CombinerInfo &CInfo, GISelValueTracking &VT,
      GISelCSEInfo *CSEInfo,
      const RISCVPostLegalizerCombinerImplRuleConfig &RuleConfig,
      const RISCVSubtarget &STI, MachineDominatorTree *MDT,
      const LegalizerInfo *LI);

  static const char *getName() { return "RISCVPostLegalizerCombiner"; }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 112-123: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp
  bool tryCombineAll(MachineInstr &I) const override;

private:
#define GET_GICOMBINER_CLASS_MEMBERS
#include "RISCVGenPostLegalizeGICombiner.inc"
#undef GET_GICOMBINER_CLASS_MEMBERS
};

#define GET_GICOMBINER_IMPL
#include "RISCVGenPostLegalizeGICombiner.inc"
#undef GET_GICOMBINER_IMPL
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 124-137: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp
RISCVPostLegalizerCombinerImpl::RISCVPostLegalizerCombinerImpl(
    MachineFunction &MF, CombinerInfo &CInfo, GISelValueTracking &VT,
    GISelCSEInfo *CSEInfo,
    const RISCVPostLegalizerCombinerImplRuleConfig &RuleConfig,
    const RISCVSubtarget &STI, MachineDominatorTree *MDT,
    const LegalizerInfo *LI)
    : Combiner(MF, CInfo, &VT, CSEInfo),
      Helper(Observer, B, /*IsPreLegalize*/ false, &VT, MDT, LI),
      RuleConfig(RuleConfig), STI(STI),
#define GET_GICOMBINER_CONSTRUCTOR_INITS
#include "RISCVGenPostLegalizeGICombiner.inc"
#undef GET_GICOMBINER_CONSTRUCTOR_INITS
{
}
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 138-147: Type declaration for RISCVPostLegalizerCombiner / RISCVPostLegalizerCombiner 的类型声明
```cpp

class RISCVPostLegalizerCombiner : public MachineFunctionPass {
public:
  static char ID;

  RISCVPostLegalizerCombiner();

  StringRef getPassName() const override {
    return "RISCVPostLegalizerCombiner";
  }
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 148-167: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp

  bool runOnMachineFunction(MachineFunction &MF) override;
  void getAnalysisUsage(AnalysisUsage &AU) const override;

private:
  RISCVPostLegalizerCombinerImplRuleConfig RuleConfig;
};
} // end anonymous namespace

void RISCVPostLegalizerCombiner::getAnalysisUsage(AnalysisUsage &AU) const {
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
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 168-184: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
}

RISCVPostLegalizerCombiner::RISCVPostLegalizerCombiner()
    : MachineFunctionPass(ID) {
  if (!RuleConfig.parseCommandLineOption())
    report_fatal_error("Invalid rule identifier");
}

bool RISCVPostLegalizerCombiner::runOnMachineFunction(MachineFunction &MF) {
  if (MF.getProperties().hasFailedISel())
    return false;
  assert(MF.getProperties().hasLegalized() && "Expected a legalized function?");
  auto *TPC = &getAnalysis<TargetPassConfig>();
  const Function &F = MF.getFunction();
  bool EnableOpt =
      MF.getTarget().getOptLevel() != CodeGenOptLevel::None && !skipFunction(F);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 185-195: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  const RISCVSubtarget &ST = MF.getSubtarget<RISCVSubtarget>();
  const auto *LI = ST.getLegalizerInfo();

  GISelValueTracking *VT =
      &getAnalysis<GISelValueTrackingAnalysisLegacy>().get(MF);
  MachineDominatorTree *MDT =
      &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
  GISelCSEAnalysisWrapper &Wrapper =
      getAnalysis<GISelCSEAnalysisWrapperPass>().getCSEWrapper();
  auto *CSEInfo = &Wrapper.get(TPC->getCSEConfig());
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 196-213: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  CombinerInfo CInfo(/*AllowIllegalOps*/ true, /*ShouldLegalizeIllegal*/ false,
                     /*LegalizerInfo*/ nullptr, EnableOpt, F.hasOptSize(),
                     F.hasMinSize());
  RISCVPostLegalizerCombinerImpl Impl(MF, CInfo, *VT, CSEInfo, RuleConfig, ST,
                                      MDT, LI);
  return Impl.combineMachineInstrs();
}

char RISCVPostLegalizerCombiner::ID = 0;
INITIALIZE_PASS_BEGIN(RISCVPostLegalizerCombiner, DEBUG_TYPE,
                      "Combine RISC-V MachineInstrs after legalization", false,
                      false)
INITIALIZE_PASS_DEPENDENCY(TargetPassConfig)
INITIALIZE_PASS_DEPENDENCY(GISelValueTrackingAnalysisLegacy)
INITIALIZE_PASS_END(RISCVPostLegalizerCombiner, DEBUG_TYPE,
                    "Combine RISC-V MachineInstrs after legalization", false,
                    false)
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 214-216: Function implementation: llvm::createRISCVPostLegalizerCombiner / 函数实现：llvm::createRISCVPostLegalizerCombiner
```cpp
FunctionPass *llvm::createRISCVPostLegalizerCombiner() {
  return new RISCVPostLegalizerCombiner();
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

## Key Concepts / 关键概念
- **GlobalISel pipeline** / **GlobalISel 流水线**
- **Operation legalization** / **操作合法化**

## Dependencies / 依赖关系
- `RISCVTargetMachine.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/GlobalISel/CSEInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/GlobalISel/Combiner.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/GlobalISel/CombinerHelper.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/GlobalISel/CombinerInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/GlobalISel/GIMatchTableExecutorImpl.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/GlobalISel/GISelValueTracking.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/GlobalISel/MachineIRBuilder.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineDominators.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineFunctionPass.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/TargetPassConfig.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/Support/FormatVariadic.h` — Directly referenced by this file. / 该文件直接引用的依赖。
