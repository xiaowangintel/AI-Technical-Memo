# GCNPreRAOptimizations.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/GCNPreRAOptimizations.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements GCNPreRAOptimizations for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 GCNPreRAOptimizations 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-32: File banner, license, and overview
```cpp
//===-- GCNPreRAOptimizations.cpp -----------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// This pass combines split register tuple initialization into a single pseudo:
///
///   undef %0.sub1:sreg_64 = S_MOV_B32 1
///   %0.sub0:sreg_64 = S_MOV_B32 2
/// =>
///   %0:sreg_64 = S_MOV_B64_IMM_PSEUDO 0x200000001
///
/// This is to allow rematerialization of a value instead of spilling. It is
/// supposed to be done after register coalescer to allow it to do its job and
/// before actual register allocation to allow rematerialization.
///
/// Right now the pass only handles 64 bit SGPRs with immediate initializers,
/// although the same shall be possible with other register classes and
/// instructions if necessary.
///
/// This pass also adds register allocation hints to COPY.
/// The hints will be post-processed by SIRegisterInfo::getRegAllocationHints.
/// When using True16, we often see COPY moving a 16-bit value between a VGPR_32
/// and a VGPR_16. If we use the VGPR_16 that corresponds to the lo16 bits of
/// the VGPR_32, the COPY can be completely eliminated.
///
//===----------------------------------------------------------------------===//

```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。

### Lines 33-63: Header dependencies and setup
```cpp
#include "GCNPreRAOptimizations.h"
#include "AMDGPU.h"
#include "GCNSubtarget.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "SIRegisterInfo.h"
#include "llvm/CodeGen/LiveIntervals.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/InitializePasses.h"

using namespace llvm;

#define DEBUG_TYPE "amdgpu-pre-ra-optimizations"

namespace {

class GCNPreRAOptimizationsImpl {
private:
  const SIInstrInfo *TII;
  const SIRegisterInfo *TRI;
  MachineRegisterInfo *MRI;
  LiveIntervals *LIS;

  bool processReg(Register Reg);
  void hintTrue16Copy(const MachineInstr &MI);
  bool optimizeBVHStack(MachineInstr &MI);

public:
  GCNPreRAOptimizationsImpl(LiveIntervals *LS) : LIS(LS) {}
  bool run(MachineFunction &MF);
};

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `GCNPreRAOptimizationsImpl`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`GCNPreRAOptimizationsImpl`。

### Lines 64-97: Registers LLVM passes
```cpp
class GCNPreRAOptimizationsLegacy : public MachineFunctionPass {
public:
  static char ID;

  GCNPreRAOptimizationsLegacy() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;

  StringRef getPassName() const override {
    return "AMDGPU Pre-RA optimizations";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<LiveIntervalsWrapperPass>();
    AU.setPreservesAll();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
};
} // End anonymous namespace.

INITIALIZE_PASS_BEGIN(GCNPreRAOptimizationsLegacy, DEBUG_TYPE,
                      "AMDGPU Pre-RA optimizations", false, false)
INITIALIZE_PASS_DEPENDENCY(LiveIntervalsWrapperPass)
INITIALIZE_PASS_END(GCNPreRAOptimizationsLegacy, DEBUG_TYPE,
                    "Pre-RA optimizations", false, false)

char GCNPreRAOptimizationsLegacy::ID = 0;

char &llvm::GCNPreRAOptimizationsID = GCNPreRAOptimizationsLegacy::ID;

FunctionPass *llvm::createGCNPreRAOptimizationsLegacyPass() {
  return new GCNPreRAOptimizationsLegacy();
}

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `GCNPreRAOptimizationsLegacy`, `MachineFunctionPass::getAnalysisUsage`, `llvm::createGCNPreRAOptimizationsLegacyPass`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`GCNPreRAOptimizationsLegacy`, `MachineFunctionPass::getAnalysisUsage`, `llvm::createGCNPreRAOptimizationsLegacyPass`。

### Lines 98-129: Implements GCNPreRAOptimizationsImpl::processReg
```cpp
bool GCNPreRAOptimizationsImpl::processReg(Register Reg) {
  MachineInstr *Def0 = nullptr;
  MachineInstr *Def1 = nullptr;
  uint64_t Init = 0;
  bool Changed = false;
  SmallSet<Register, 32> ModifiedRegs;
  bool IsAGPRDst = TRI->isAGPRClass(MRI->getRegClass(Reg));

  for (MachineInstr &I : MRI->def_instructions(Reg)) {
    switch (I.getOpcode()) {
    default:
      return false;
    case AMDGPU::V_ACCVGPR_WRITE_B32_e64:
      break;
    case AMDGPU::COPY: {
      // Some subtargets cannot do an AGPR to AGPR copy directly, and need an
      // intermdiate temporary VGPR register. Try to find the defining
      // accvgpr_write to avoid temporary registers.

      if (!IsAGPRDst)
        return false;

      Register SrcReg = I.getOperand(1).getReg();

      if (!SrcReg.isVirtual())
        break;

      // Check if source of copy is from another AGPR.
      bool IsAGPRSrc = TRI->isAGPRClass(MRI->getRegClass(SrcReg));
      if (!IsAGPRSrc)
        break;

```
**EN:** This section contains concrete logic for GCNPreRAOptimizationsImpl::processReg. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNPreRAOptimizationsImpl::processReg`.
**CN:** 本节包含与 GCNPreRAOptimizationsImpl::processReg 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNPreRAOptimizationsImpl::processReg`。

### Lines 130-156: Conditional logic and checks
```cpp
      // def_instructions() does not look at subregs so it may give us a
      // different instruction that defines the same vreg but different subreg
      // so we have to manually check subreg.
      Register SrcSubReg = I.getOperand(1).getSubReg();
      for (auto &Def : MRI->def_instructions(SrcReg)) {
        if (SrcSubReg != Def.getOperand(0).getSubReg())
          continue;

        if (Def.getOpcode() == AMDGPU::V_ACCVGPR_WRITE_B32_e64) {
          const MachineOperand &DefSrcMO = Def.getOperand(1);

          // Immediates are not an issue and can be propagated in
          // postrapseudos pass. Only handle cases where defining
          // accvgpr_write source is a vreg.
          if (DefSrcMO.isReg() && DefSrcMO.getReg().isVirtual()) {
            // Propagate source reg of accvgpr write to this copy instruction
            I.getOperand(1).setReg(DefSrcMO.getReg());
            I.getOperand(1).setSubReg(DefSrcMO.getSubReg());

            // Reg uses were changed, collect unique set of registers to update
            // live intervals at the end.
            ModifiedRegs.insert(DefSrcMO.getReg());
            ModifiedRegs.insert(SrcReg);

            Changed = true;
          }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 157-187: Switch-based control flow
```cpp
          // Found the defining accvgpr_write, stop looking any further.
          break;
        }
      }
      break;
    }
    case AMDGPU::S_MOV_B32:
      if (I.getOperand(0).getReg() != Reg || !I.getOperand(1).isImm() ||
          I.getNumOperands() != 2)
        return false;

      switch (I.getOperand(0).getSubReg()) {
      default:
        return false;
      case AMDGPU::sub0:
        if (Def0)
          return false;
        Def0 = &I;
        Init |= Lo_32(I.getOperand(1).getImm());
        break;
      case AMDGPU::sub1:
        if (Def1)
          return false;
        Def1 = &I;
        Init |= static_cast<uint64_t>(I.getOperand(1).getImm()) << 32;
        break;
      }
      break;
    }
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 188-216: Conditional logic and checks
```cpp
  // For AGPR reg, check if live intervals need to be updated.
  if (IsAGPRDst) {
    if (Changed) {
      for (Register RegToUpdate : ModifiedRegs) {
        LIS->removeInterval(RegToUpdate);
        LIS->createAndComputeVirtRegInterval(RegToUpdate);
      }
    }

    return Changed;
  }

  // For SGPR reg, check if we can combine instructions.
  if (!Def0 || !Def1 || Def0->getParent() != Def1->getParent())
    return Changed;

  LLVM_DEBUG(dbgs() << "Combining:\n  " << *Def0 << "  " << *Def1
                    << "    =>\n");

  if (SlotIndex::isEarlierInstr(LIS->getInstructionIndex(*Def1),
                                LIS->getInstructionIndex(*Def0)))
    std::swap(Def0, Def1);

  LIS->RemoveMachineInstrFromMaps(*Def0);
  LIS->RemoveMachineInstrFromMaps(*Def1);
  auto NewI = BuildMI(*Def0->getParent(), *Def0, Def0->getDebugLoc(),
                      TII->get(AMDGPU::S_MOV_B64_IMM_PSEUDO), Reg)
                  .addImm(Init);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `SlotIndex::isEarlierInstr`, `std::swap`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`SlotIndex::isEarlierInstr`, `std::swap`。

### Lines 217-242: Declares eraseFromParent
```cpp
  Def0->eraseFromParent();
  Def1->eraseFromParent();
  LIS->InsertMachineInstrInMaps(*NewI);
  LIS->removeInterval(Reg);
  LIS->createAndComputeVirtRegInterval(Reg);

  LLVM_DEBUG(dbgs() << "  " << *NewI);

  return true;
}

bool GCNPreRAOptimizationsLegacy::runOnMachineFunction(MachineFunction &MF) {
  if (skipFunction(MF.getFunction()))
    return false;
  LiveIntervals *LIS = &getAnalysis<LiveIntervalsWrapperPass>().getLIS();
  return GCNPreRAOptimizationsImpl(LIS).run(MF);
}

PreservedAnalyses
GCNPreRAOptimizationsPass::run(MachineFunction &MF,
                               MachineFunctionAnalysisManager &MFAM) {
  LiveIntervals *LIS = &MFAM.getResult<LiveIntervalsAnalysis>(MF);
  GCNPreRAOptimizationsImpl(LIS).run(MF);
  return PreservedAnalyses::all();
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `GCNPreRAOptimizationsLegacy::runOnMachineFunction`, `GCNPreRAOptimizationsPass::run`, `PreservedAnalyses::all`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`GCNPreRAOptimizationsLegacy::runOnMachineFunction`, `GCNPreRAOptimizationsPass::run`, `PreservedAnalyses::all`。

### Lines 243-267: Implements GCNPreRAOptimizationsImpl::hintTrue16Copy
```cpp
void GCNPreRAOptimizationsImpl::hintTrue16Copy(const MachineInstr &MI) {
  Register Dst = MI.getOperand(0).getReg();
  Register Src = MI.getOperand(1).getReg();
  const TargetRegisterClass *DstRC = TRI->getRegClassForReg(*MRI, Dst);
  bool IsDst16Bit = AMDGPU::VGPR_16RegClass.hasSubClassEq(DstRC);
  if (Dst.isVirtual() && IsDst16Bit && Src.isPhysical() &&
      TRI->getRegClassForReg(*MRI, Src) == &AMDGPU::VGPR_32RegClass)
    MRI->setRegAllocationHint(Dst, 0, TRI->getSubReg(Src, AMDGPU::lo16));
  if (Src.isVirtual() && MRI->getRegClass(Src) == &AMDGPU::VGPR_16RegClass &&
      Dst.isPhysical() && DstRC == &AMDGPU::VGPR_32RegClass)
    MRI->setRegAllocationHint(Src, 0, TRI->getSubReg(Dst, AMDGPU::lo16));
  if (!Dst.isVirtual() || !Src.isVirtual())
    return;
  if (MRI->getRegClass(Dst) == &AMDGPU::VGPR_32RegClass &&
      MRI->getRegClass(Src) == &AMDGPU::VGPR_16RegClass) {
    MRI->setRegAllocationHint(Dst, AMDGPURI::Size32, Src);
    MRI->setRegAllocationHint(Src, AMDGPURI::Size16, Dst);
  }
  if (IsDst16Bit && MRI->getRegClass(Src) == &AMDGPU::VGPR_32RegClass)
    MRI->setRegAllocationHint(Dst, AMDGPURI::Size16, Src);
}

bool GCNPreRAOptimizationsImpl::optimizeBVHStack(MachineInstr &MI) {
  SmallVector<Register, 2> UseRegs;

```
**EN:** This section contains concrete logic for GCNPreRAOptimizationsImpl::hintTrue16Copy. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNPreRAOptimizationsImpl::hintTrue16Copy`, `GCNPreRAOptimizationsImpl::optimizeBVHStack`.
**CN:** 本节包含与 GCNPreRAOptimizationsImpl::hintTrue16Copy 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNPreRAOptimizationsImpl::hintTrue16Copy`, `GCNPreRAOptimizationsImpl::optimizeBVHStack`。

### Lines 268-301: Conditional logic and checks
```cpp
  // Find BVH sources for this DS_BVH_STACK instruction.
  auto CheckUse = [&](MachineOperand &Use) {
    Register Reg = Use.getReg();
    for (const MachineInstr &Src : MRI->def_instructions(Reg)) {
      if (!SIInstrInfo::isImage(Src))
        continue;
      const AMDGPU::MIMGInfo *Info = AMDGPU::getMIMGInfo(Src.getOpcode());
      const AMDGPU::MIMGBaseOpcodeInfo *BaseInfo =
          AMDGPU::getMIMGBaseOpcodeInfo(Info->BaseOpcode);
      if (!BaseInfo->BVH)
        continue;
      UseRegs.push_back(Reg);
      break;
    }
  };
  CheckUse(*TII->getNamedOperand(MI, AMDGPU::OpName::data0));
  CheckUse(*TII->getNamedOperand(MI, AMDGPU::OpName::data1));

  if (UseRegs.empty())
    return false;

  // Add implicit uses for entire BVH source registers.
  // This avoids partial reallocation of register which could
  // introduce a premature s_wait_bvhcnt.
  for (Register Reg : UseRegs) {
    MI.addOperand(MachineOperand::CreateReg(Reg, false, true));
    LIS->removeInterval(Reg);
    LIS->createAndComputeVirtRegInterval(Reg);
  }
  LLVM_DEBUG(dbgs() << "Added implicit uses to: " << MI);

  return true;
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `SIInstrInfo::isImage`, `AMDGPU::getMIMGInfo`, `AMDGPU::getMIMGBaseOpcodeInfo`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`SIInstrInfo::isImage`, `AMDGPU::getMIMGInfo`, `AMDGPU::getMIMGBaseOpcodeInfo`。

### Lines 302-327: Implements GCNPreRAOptimizationsImpl::run
```cpp
bool GCNPreRAOptimizationsImpl::run(MachineFunction &MF) {
  const GCNSubtarget &ST = MF.getSubtarget<GCNSubtarget>();
  TII = ST.getInstrInfo();
  MRI = &MF.getRegInfo();
  TRI = ST.getRegisterInfo();

  bool Changed = false;

  for (unsigned I = 0, E = MRI->getNumVirtRegs(); I != E; ++I) {
    Register Reg = Register::index2VirtReg(I);
    if (!LIS->hasInterval(Reg))
      continue;
    const TargetRegisterClass *RC = MRI->getRegClass(Reg);
    if ((RC->MC->getSizeInBits() != 64 || !TRI->isSGPRClass(RC)) &&
        (ST.hasGFX90AInsts() || !TRI->isAGPRClass(RC)))
      continue;

    Changed |= processReg(Reg);
  }

  const bool HasBVHStack = ST.hasBVHDualAndBVH8Insts();
  const bool HasRealTrue16 = ST.useRealTrue16Insts();

  if (!HasRealTrue16 && !HasBVHStack)
    return Changed;

```
**EN:** This section contains concrete logic for GCNPreRAOptimizationsImpl::run. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNPreRAOptimizationsImpl::run`, `Register::index2VirtReg`.
**CN:** 本节包含与 GCNPreRAOptimizationsImpl::run 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNPreRAOptimizationsImpl::run`, `Register::index2VirtReg`。

### Lines 328-347: Conditional logic and checks
```cpp
  for (MachineBasicBlock &MBB : MF) {
    for (MachineInstr &MI : MBB) {
      // Add RA hints to improve True16 COPY elimination.
      if (HasRealTrue16 && MI.getOpcode() == AMDGPU::COPY) {
        hintTrue16Copy(MI);
        continue;
      }
      // Add implicit uses to avoid early wait on intersect ray instructions.
      if (HasBVHStack &&
          (MI.getOpcode() == AMDGPU::DS_BVH_STACK_RTN_B32 ||
           MI.getOpcode() == AMDGPU::DS_BVH_STACK_PUSH8_POP1_RTN_B32 ||
           MI.getOpcode() == AMDGPU::DS_BVH_STACK_PUSH8_POP2_RTN_B64)) {
        Changed |= optimizeBVHStack(MI);
        continue;
      }
    }
  }

  return Changed;
}
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `GCNPreRAOptimizationsImpl`, `GCNPreRAOptimizationsLegacy`, `MachineFunctionPass::getAnalysisUsage`, `llvm::createGCNPreRAOptimizationsLegacyPass`, `GCNPreRAOptimizationsImpl::processReg`, `SlotIndex::isEarlierInstr`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"GCNPreRAOptimizations.h"`
- `"AMDGPU.h"`
- `"GCNSubtarget.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"SIRegisterInfo.h"`
- `"llvm/CodeGen/LiveIntervals.h"`
- `"llvm/CodeGen/MachineFunctionPass.h"`
- `"llvm/InitializePasses.h"`
