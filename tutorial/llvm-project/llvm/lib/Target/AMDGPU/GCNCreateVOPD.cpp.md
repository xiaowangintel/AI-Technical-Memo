# GCNCreateVOPD.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/GCNCreateVOPD.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements GCNCreateVOPD for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 GCNCreateVOPD 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-32: File banner, includes, and setup
```cpp
//===- GCNCreateVOPD.cpp - Create VOPD Instructions ----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// Combine VALU pairs into VOPD instructions
/// Only works on wave32
/// Has register requirements, we reject creating VOPD if the requirements are
/// not met.
/// shouldCombineVOPD mutator in postRA machine scheduler puts candidate
/// instructions for VOPD back-to-back
///
//
//===----------------------------------------------------------------------===//

#include "AMDGPU.h"
#include "GCNSubtarget.h"
#include "GCNVOPDUtils.h"
#include "SIInstrInfo.h"
#include "Utils/AMDGPUBaseInfo.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/CodeGen/MachinePassManager.h"
#include "llvm/Support/Debug.h"

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 33-56: Preprocessor guards and macros
```cpp
#define DEBUG_TYPE "gcn-create-vopd"
STATISTIC(NumVOPDCreated, "Number of VOPD Insts Created.");

using namespace llvm;

namespace {

class GCNCreateVOPD {
private:
  class VOPDCombineInfo {
  public:
    VOPDCombineInfo() = default;
    VOPDCombineInfo(MachineInstr *First, MachineInstr *Second,
                    bool VOPD3 = false)
        : FirstMI(First), SecondMI(Second), IsVOPD3(VOPD3) {}

    MachineInstr *FirstMI;
    MachineInstr *SecondMI;
    bool IsVOPD3;
  };

public:
  const GCNSubtarget *ST = nullptr;

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `GCNCreateVOPD`, `VOPDCombineInfo`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`GCNCreateVOPD`, `VOPDCombineInfo`。

### Lines 57-83: Defines doReplace
```cpp
  bool doReplace(const SIInstrInfo *SII, VOPDCombineInfo &CI) {
    auto *FirstMI = CI.FirstMI;
    auto *SecondMI = CI.SecondMI;
    unsigned Opc1 = FirstMI->getOpcode();
    unsigned Opc2 = SecondMI->getOpcode();
    unsigned EncodingFamily =
        AMDGPU::getVOPDEncodingFamily(SII->getSubtarget());
    int NewOpcode = AMDGPU::getVOPDFull(AMDGPU::getVOPDOpcode(Opc1, CI.IsVOPD3),
                                        AMDGPU::getVOPDOpcode(Opc2, CI.IsVOPD3),
                                        EncodingFamily, CI.IsVOPD3);
    assert(NewOpcode != -1 &&
           "Should have previously determined this as a possible VOPD\n");

    auto VOPDInst = BuildMI(*FirstMI->getParent(), FirstMI,
                            FirstMI->getDebugLoc(), SII->get(NewOpcode))
                        .setMIFlags(FirstMI->getFlags() | SecondMI->getFlags());

    namespace VOPD = AMDGPU::VOPD;
    MachineInstr *MI[] = {FirstMI, SecondMI};
    auto InstInfo =
        AMDGPU::getVOPDInstInfo(FirstMI->getDesc(), SecondMI->getDesc());

    for (auto CompIdx : VOPD::COMPONENTS) {
      auto MCOprIdx = InstInfo[CompIdx].getIndexOfDstInMCOperands();
      VOPDInst.add(MI[CompIdx]->getOperand(MCOprIdx));
    }

```
**EN:** This section contains concrete logic for doReplace. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPU::getVOPDEncodingFamily`, `AMDGPU::getVOPDFull`, `AMDGPU::getVOPDOpcode`.
**CN:** 本节包含与 doReplace 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPU::getVOPDEncodingFamily`, `AMDGPU::getVOPDFull`, `AMDGPU::getVOPDOpcode`。

### Lines 84-115: Conditional logic and checks
```cpp
    const AMDGPU::OpName Mods[2][3] = {
        {AMDGPU::OpName::src0X_modifiers, AMDGPU::OpName::vsrc1X_modifiers,
         AMDGPU::OpName::vsrc2X_modifiers},
        {AMDGPU::OpName::src0Y_modifiers, AMDGPU::OpName::vsrc1Y_modifiers,
         AMDGPU::OpName::vsrc2Y_modifiers}};
    const AMDGPU::OpName SrcMods[3] = {AMDGPU::OpName::src0_modifiers,
                                       AMDGPU::OpName::src1_modifiers,
                                       AMDGPU::OpName::src2_modifiers};
    const unsigned VOPDOpc = VOPDInst->getOpcode();

    for (auto CompIdx : VOPD::COMPONENTS) {
      auto CompSrcOprNum = InstInfo[CompIdx].getCompSrcOperandsNum();
      bool IsVOP3 = SII->isVOP3(*MI[CompIdx]);
      for (unsigned CompSrcIdx = 0; CompSrcIdx < CompSrcOprNum; ++CompSrcIdx) {
        if (AMDGPU::hasNamedOperand(VOPDOpc, Mods[CompIdx][CompSrcIdx])) {
          const MachineOperand *Mod =
              SII->getNamedOperand(*MI[CompIdx], SrcMods[CompSrcIdx]);
          VOPDInst.addImm(Mod ? Mod->getImm() : 0);
        }
        auto MCOprIdx =
            InstInfo[CompIdx].getIndexOfSrcInMCOperands(CompSrcIdx, IsVOP3);
        VOPDInst.add(MI[CompIdx]->getOperand(MCOprIdx));
      }
      if (MI[CompIdx]->getOpcode() == AMDGPU::V_CNDMASK_B32_e32 && CI.IsVOPD3)
        VOPDInst.addReg(AMDGPU::VCC_LO);
    }

    if (CI.IsVOPD3) {
      if (unsigned BitOp2 = AMDGPU::getBitOp2(Opc2))
        VOPDInst.addImm(BitOp2);
    }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::hasNamedOperand`, `AMDGPU::getBitOp2`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::hasNamedOperand`, `AMDGPU::getBitOp2`。

### Lines 116-140: Conditional logic and checks
```cpp
    SII->fixImplicitOperands(*VOPDInst);
    for (auto CompIdx : VOPD::COMPONENTS)
      VOPDInst.copyImplicitOps(*MI[CompIdx]);

    LLVM_DEBUG(dbgs() << "VOPD Fused: " << *VOPDInst << " from\tX: "
                      << *CI.FirstMI << "\tY: " << *CI.SecondMI << "\n");

    for (auto CompIdx : VOPD::COMPONENTS)
      MI[CompIdx]->eraseFromParent();

    ++NumVOPDCreated;
    return true;
  }

  bool run(MachineFunction &MF) {
    ST = &MF.getSubtarget<GCNSubtarget>();
    if (!AMDGPU::hasVOPD(*ST) || !ST->isWave32())
      return false;
    LLVM_DEBUG(dbgs() << "CreateVOPD Pass:\n");

    const SIInstrInfo *SII = ST->getInstrInfo();
    bool Changed = false;

    SmallVector<VOPDCombineInfo> ReplaceCandidates;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::hasVOPD`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::hasVOPD`。

### Lines 141-171: Declares class GCNCreateVOPDLegacy
```cpp
    for (auto &MBB : MF) {
      auto MII = MBB.begin(), E = MBB.end();
      while (MII != E) {
        auto *FirstMI = &*MII;
        MII = next_nodbg(MII, MBB.end());
        if (MII == MBB.end())
          break;
        if (FirstMI->isDebugInstr())
          continue;
        auto *SecondMI = &*MII;

        if (auto Match = tryMatchVOPDPair(*SII, *FirstMI, *SecondMI)) {
          ReplaceCandidates.push_back(
              VOPDCombineInfo(Match->MIX, Match->MIY, Match->IsVOPD3));
          ++MII;
        }
      }
    }
    for (auto &CI : ReplaceCandidates) {
      Changed |= doReplace(SII, CI);
    }

    return Changed;
  }
};

class GCNCreateVOPDLegacy : public MachineFunctionPass {
public:
  static char ID;
  GCNCreateVOPDLegacy() : MachineFunctionPass(ID) {}

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `GCNCreateVOPDLegacy`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`GCNCreateVOPDLegacy`。

### Lines 172-203: Registers LLVM passes
```cpp
  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  StringRef getPassName() const override {
    return "GCN Create VOPD Instructions";
  }
  bool runOnMachineFunction(MachineFunction &MF) override {
    if (skipFunction(MF.getFunction()))
      return false;

    return GCNCreateVOPD().run(MF);
  }
};

} // namespace

PreservedAnalyses
llvm::GCNCreateVOPDPass::run(MachineFunction &MF,
                             MachineFunctionAnalysisManager &AM) {
  if (!GCNCreateVOPD().run(MF))
    return PreservedAnalyses::all();
  return getMachineFunctionPassPreservedAnalyses().preserveSet<CFGAnalyses>();
}

char GCNCreateVOPDLegacy::ID = 0;

char &llvm::GCNCreateVOPDID = GCNCreateVOPDLegacy::ID;

INITIALIZE_PASS(GCNCreateVOPDLegacy, DEBUG_TYPE, "GCN Create VOPD Instructions",
                false, false)
```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `MachineFunctionPass::getAnalysisUsage`, `GCNCreateVOPDPass::run`, `PreservedAnalyses::all`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`MachineFunctionPass::getAnalysisUsage`, `GCNCreateVOPDPass::run`, `PreservedAnalyses::all`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `GCNCreateVOPD`, `VOPDCombineInfo`, `GCNCreateVOPDLegacy`, `AMDGPU::getVOPDEncodingFamily`, `AMDGPU::getVOPDFull`, `AMDGPU::getVOPDOpcode`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; scheduling / 调度
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPU.h"`
- `"GCNSubtarget.h"`
- `"GCNVOPDUtils.h"`
- `"SIInstrInfo.h"`
- `"Utils/AMDGPUBaseInfo.h"`
- `"llvm/ADT/SmallVector.h"`
- `"llvm/ADT/Statistic.h"`
- `"llvm/CodeGen/MachineBasicBlock.h"`
- `"llvm/CodeGen/MachineInstr.h"`
- `"llvm/CodeGen/MachineOperand.h"`
- `"llvm/CodeGen/MachinePassManager.h"`
- `"llvm/Support/Debug.h"`
