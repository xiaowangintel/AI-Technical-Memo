# R600Packetizer.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/R600Packetizer.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements R600Packetizer for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 R600Packetizer 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-31: File banner, includes, and setup
```cpp
//===----- R600Packetizer.cpp - VLIW packetizer ---------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// This pass implements instructions packetization for R600. It unsets isLast
/// bit of instructions inside a bundle and substitutes src register with
/// PreviousVector when applicable.
//
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/R600MCTargetDesc.h"
#include "R600.h"
#include "R600Subtarget.h"
#include "llvm/CodeGen/DFAPacketizer.h"
#include "llvm/CodeGen/MachineDominators.h"
#include "llvm/CodeGen/MachineLoopInfo.h"
#include "llvm/CodeGen/ScheduleDAG.h"

using namespace llvm;

#define DEBUG_TYPE "packets"

namespace {

class R600Packetizer : public MachineFunctionPass {

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `R600Packetizer`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`R600Packetizer`。

### Lines 32-60: Declares class R600PacketizerList
```cpp
public:
  static char ID;
  R600Packetizer() : MachineFunctionPass(ID) {}

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    AU.addRequired<MachineDominatorTreeWrapperPass>();
    AU.addPreserved<MachineDominatorTreeWrapperPass>();
    AU.addRequired<MachineLoopInfoWrapperPass>();
    AU.addPreserved<MachineLoopInfoWrapperPass>();
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  StringRef getPassName() const override { return "R600 Packetizer"; }

  bool runOnMachineFunction(MachineFunction &Fn) override;
};

class R600PacketizerList : public VLIWPacketizerList {
private:
  const R600InstrInfo *TII;
  const R600RegisterInfo &TRI;
  bool VLIW5;
  bool ConsideredInstUsesAlreadyWrittenVectorElement;

  unsigned getSlot(const MachineInstr &MI) const {
    return TRI.getHWRegChan(MI.getOperand(0).getReg());
  }

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `R600PacketizerList`, `MachineFunctionPass::getAnalysisUsage`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`R600PacketizerList`, `MachineFunctionPass::getAnalysisUsage`。

### Lines 61-94: Defines getPreviousVector
```cpp
  /// \returns register to PV chan mapping for bundle/single instructions that
  /// immediately precedes I.
  DenseMap<unsigned, unsigned> getPreviousVector(MachineBasicBlock::iterator I)
      const {
    DenseMap<unsigned, unsigned> Result;
    I--;
    if (!TII->isALUInstr(I->getOpcode()) && !I->isBundle())
      return Result;
    MachineBasicBlock::instr_iterator BI = I.getInstrIterator();
    if (I->isBundle())
      BI++;
    int LastDstChan = -1;
    do {
      bool isTrans = false;
      int BISlot = getSlot(*BI);
      if (LastDstChan >= BISlot)
        isTrans = true;
      LastDstChan = BISlot;
      if (TII->isPredicated(*BI))
        continue;
      int OperandIdx = TII->getOperandIdx(BI->getOpcode(), R600::OpName::write);
      if (OperandIdx > -1 && BI->getOperand(OperandIdx).getImm() == 0)
        continue;
      int DstIdx = TII->getOperandIdx(BI->getOpcode(), R600::OpName::dst);
      if (DstIdx == -1) {
        continue;
      }
      Register Dst = BI->getOperand(DstIdx).getReg();
      if (isTrans || TII->isTransOnly(*BI)) {
        Result[Dst] = R600::PS;
        continue;
      }
      if (BI->getOpcode() == R600::DOT4_r600 ||
          BI->getOpcode() == R600::DOT4_eg) {
```
**EN:** This section contains concrete logic for getPreviousVector. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getPreviousVector 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 95-122: Switch-based control flow
```cpp
        Result[Dst] = R600::PV_X;
        continue;
      }
      if (Dst == R600::OQAP) {
        continue;
      }
      unsigned PVReg = 0;
      switch (TRI.getHWRegChan(Dst)) {
      case 0:
        PVReg = R600::PV_X;
        break;
      case 1:
        PVReg = R600::PV_Y;
        break;
      case 2:
        PVReg = R600::PV_Z;
        break;
      case 3:
        PVReg = R600::PV_W;
        break;
      default:
        llvm_unreachable("Invalid Chan");
      }
      Result[Dst] = PVReg;
    } while ((++BI)->isBundledWithPred());
    return Result;
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 123-151: Defines substitutePV
```cpp
  void substitutePV(MachineInstr &MI, const DenseMap<unsigned, unsigned> &PVs)
      const {
    const R600::OpName Ops[] = {R600::OpName::src0, R600::OpName::src1,
                                R600::OpName::src2};
    for (R600::OpName Op : Ops) {
      int OperandIdx = TII->getOperandIdx(MI.getOpcode(), Op);
      if (OperandIdx < 0)
        continue;
      Register Src = MI.getOperand(OperandIdx).getReg();
      const auto It = PVs.find(Src);
      if (It != PVs.end())
        MI.getOperand(OperandIdx).setReg(It->second);
    }
  }
public:
  // Ctor.
  R600PacketizerList(MachineFunction &MF, const R600Subtarget &ST,
                     MachineLoopInfo &MLI)
      : VLIWPacketizerList(MF, MLI, nullptr),
        TII(ST.getInstrInfo()),
        TRI(TII->getRegisterInfo()) {
    VLIW5 = !ST.hasCaymanISA();
  }

  // initPacketizerState - initialize some internal flags.
  void initPacketizerState() override {
    ConsideredInstUsesAlreadyWrittenVectorElement = false;
  }

```
**EN:** This section contains concrete logic for substitutePV. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 substitutePV 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 152-171: Defines ignorePseudoInstruction
```cpp
  // ignorePseudoInstruction - Ignore bundling of pseudo instructions.
  bool ignorePseudoInstruction(const MachineInstr &MI,
                               const MachineBasicBlock *MBB) override {
    return false;
  }

  // isSoloInstruction - return true if instruction MI can not be packetized
  // with any other instruction, which means that MI itself is a packet.
  bool isSoloInstruction(const MachineInstr &MI) override {
    if (TII->isVector(MI))
      return true;
    if (!TII->isALUInstr(MI.getOpcode()))
      return true;
    if (MI.getOpcode() == R600::GROUP_BARRIER)
      return true;
    // XXX: This can be removed once the packetizer properly handles all the
    // LDS instruction group restrictions.
    return TII->isLDSInstr(MI.getOpcode());
  }

```
**EN:** This section contains concrete logic for ignorePseudoInstruction. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 ignorePseudoInstruction 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 172-205: Defines isLegalToPacketizeTogether
```cpp
  // isLegalToPacketizeTogether - Is it legal to packetize SUI and SUJ
  // together.
  bool isLegalToPacketizeTogether(SUnit *SUI, SUnit *SUJ) override {
    MachineInstr *MII = SUI->getInstr(), *MIJ = SUJ->getInstr();
    if (getSlot(*MII) == getSlot(*MIJ))
      ConsideredInstUsesAlreadyWrittenVectorElement = true;
    // Does MII and MIJ share the same pred_sel ?
    int OpI = TII->getOperandIdx(MII->getOpcode(), R600::OpName::pred_sel),
        OpJ = TII->getOperandIdx(MIJ->getOpcode(), R600::OpName::pred_sel);
    Register PredI = (OpI > -1)?MII->getOperand(OpI).getReg() : Register(),
      PredJ = (OpJ > -1)?MIJ->getOperand(OpJ).getReg() : Register();
    if (PredI != PredJ)
      return false;
    if (SUJ->isSucc(SUI)) {
      for (const SDep &Dep : SUJ->Succs) {
        if (Dep.getSUnit() != SUI)
          continue;
        if (Dep.getKind() == SDep::Anti)
          continue;
        if (Dep.getKind() == SDep::Output)
          if (MII->getOperand(0).getReg() != MIJ->getOperand(0).getReg())
            continue;
        return false;
      }
    }

    bool ARDef =
        TII->definesAddressRegister(*MII) || TII->definesAddressRegister(*MIJ);
    bool ARUse =
        TII->usesAddressRegister(*MII) || TII->usesAddressRegister(*MIJ);

    return !ARDef || !ARUse;
  }

```
**EN:** This section contains concrete logic for isLegalToPacketizeTogether. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 isLegalToPacketizeTogether 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 206-239: Defines isLegalToPruneDependencies
```cpp
  // isLegalToPruneDependencies - Is it legal to prune dependency between SUI
  // and SUJ.
  bool isLegalToPruneDependencies(SUnit *SUI, SUnit *SUJ) override {
    return false;
  }

  void setIsLastBit(MachineInstr *MI, unsigned Bit) const {
    unsigned LastOp = TII->getOperandIdx(MI->getOpcode(), R600::OpName::last);
    MI->getOperand(LastOp).setImm(Bit);
  }

  bool isBundlableWithCurrentPMI(MachineInstr &MI,
                                 const DenseMap<unsigned, unsigned> &PV,
                                 std::vector<R600InstrInfo::BankSwizzle> &BS,
                                 bool &isTransSlot) {
    isTransSlot = TII->isTransOnly(MI);
    assert (!isTransSlot || VLIW5);

    // Is the dst reg sequence legal ?
    if (!isTransSlot && !CurrentPacketMIs.empty()) {
      if (getSlot(MI) <= getSlot(*CurrentPacketMIs.back())) {
        if (ConsideredInstUsesAlreadyWrittenVectorElement &&
            !TII->isVectorOnly(MI) && VLIW5) {
          isTransSlot = true;
          LLVM_DEBUG({
            dbgs() << "Considering as Trans Inst :";
            MI.dump();
          });
        }
        else
          return false;
      }
    }

```
**EN:** This section contains concrete logic for isLegalToPruneDependencies. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 isLegalToPruneDependencies 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 240-273: Conditional logic and checks
```cpp
    // Are the Constants limitations met ?
    CurrentPacketMIs.push_back(&MI);
    if (!TII->fitsConstReadLimitations(CurrentPacketMIs)) {
      LLVM_DEBUG({
        dbgs() << "Couldn't pack :\n";
        MI.dump();
        dbgs() << "with the following packets :\n";
        for (unsigned i = 0, e = CurrentPacketMIs.size() - 1; i < e; i++) {
          CurrentPacketMIs[i]->dump();
          dbgs() << "\n";
        }
        dbgs() << "because of Consts read limitations\n";
      });
      CurrentPacketMIs.pop_back();
      return false;
    }

    // Is there a BankSwizzle set that meet Read Port limitations ?
    if (!TII->fitsReadPortLimitations(CurrentPacketMIs,
            PV, BS, isTransSlot)) {
      LLVM_DEBUG({
        dbgs() << "Couldn't pack :\n";
        MI.dump();
        dbgs() << "with the following packets :\n";
        for (unsigned i = 0, e = CurrentPacketMIs.size() - 1; i < e; i++) {
          CurrentPacketMIs[i]->dump();
          dbgs() << "\n";
        }
        dbgs() << "because of Read port limitations\n";
      });
      CurrentPacketMIs.pop_back();
      return false;
    }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 274-307: Defines addToPacket
```cpp
    // We cannot read LDS source registers from the Trans slot.
    if (isTransSlot && TII->readsLDSSrcReg(MI))
      return false;

    CurrentPacketMIs.pop_back();
    return true;
  }

  MachineBasicBlock::iterator addToPacket(MachineInstr &MI) override {
    MachineBasicBlock::iterator FirstInBundle =
        CurrentPacketMIs.empty() ? &MI : CurrentPacketMIs.front();
    const DenseMap<unsigned, unsigned> &PV =
        getPreviousVector(FirstInBundle);
    std::vector<R600InstrInfo::BankSwizzle> BS;
    bool isTransSlot;

    if (isBundlableWithCurrentPMI(MI, PV, BS, isTransSlot)) {
      for (unsigned i = 0, e = CurrentPacketMIs.size(); i < e; i++) {
        MachineInstr *MI = CurrentPacketMIs[i];
        unsigned Op = TII->getOperandIdx(MI->getOpcode(),
            R600::OpName::bank_swizzle);
        MI->getOperand(Op).setImm(BS[i]);
      }
      unsigned Op =
          TII->getOperandIdx(MI.getOpcode(), R600::OpName::bank_swizzle);
      MI.getOperand(Op).setImm(BS.back());
      if (!CurrentPacketMIs.empty())
        setIsLastBit(CurrentPacketMIs.back(), 0);
      substitutePV(MI, PV);
      MachineBasicBlock::iterator It = VLIWPacketizerList::addToPacket(MI);
      if (isTransSlot) {
        endPacket(std::next(It)->getParent(), std::next(It));
      }
      return It;
```
**EN:** This section contains concrete logic for addToPacket. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `VLIWPacketizerList::addToPacket`, `std::next`.
**CN:** 本节包含与 addToPacket 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`VLIWPacketizerList::addToPacket`, `std::next`。

### Lines 308-336: Implements R600Packetizer::runOnMachineFunction
```cpp
    }
    endPacket(MI.getParent(), MI);
    if (TII->isTransOnly(MI))
      return MI;
    return VLIWPacketizerList::addToPacket(MI);
  }
};

bool R600Packetizer::runOnMachineFunction(MachineFunction &Fn) {
  const R600Subtarget &ST = Fn.getSubtarget<R600Subtarget>();
  const R600InstrInfo *TII = ST.getInstrInfo();

  MachineLoopInfo &MLI = getAnalysis<MachineLoopInfoWrapperPass>().getLI();

  const InstrItineraryData *II = ST.getInstrItineraryData();
  // If there is no itineraries information, abandon.
  if (II->Itineraries == nullptr)
    return false;

  // Instantiate the packetizer.
  R600PacketizerList Packetizer(Fn, ST, MLI);

  // DFA state table should not be empty.
  assert(Packetizer.getResourceTracker() && "Empty DFA table!");
  assert(Packetizer.getResourceTracker()->getInstrItins());

  if (Packetizer.getResourceTracker()->getInstrItins()->isEmpty())
    return false;

```
**EN:** This section contains concrete logic for R600Packetizer::runOnMachineFunction. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `VLIWPacketizerList::addToPacket`, `R600Packetizer::runOnMachineFunction`.
**CN:** 本节包含与 R600Packetizer::runOnMachineFunction 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`VLIWPacketizerList::addToPacket`, `R600Packetizer::runOnMachineFunction`。

### Lines 337-370: Conditional logic and checks
```cpp
  //
  // Loop over all basic blocks and remove KILL pseudo-instructions
  // These instructions confuse the dependence analysis. Consider:
  // D0 = ...   (Insn 0)
  // R0 = KILL R0, D0 (Insn 1)
  // R0 = ... (Insn 2)
  // Here, Insn 1 will result in the dependence graph not emitting an output
  // dependence between Insn 0 and Insn 2. This can lead to incorrect
  // packetization
  //
  for (MachineBasicBlock &MBB : Fn) {
    for (MachineInstr &MI : llvm::make_early_inc_range(MBB)) {
      if (MI.isKill() || MI.getOpcode() == R600::IMPLICIT_DEF ||
          (MI.getOpcode() == R600::CF_ALU && !MI.getOperand(8).getImm()))
        MBB.erase(MI);
    }
  }

  // Loop over all of the basic blocks.
  for (MachineFunction::iterator MBB = Fn.begin(), MBBe = Fn.end();
       MBB != MBBe; ++MBB) {
    // Find scheduling regions and schedule / packetize each region.
    unsigned RemainingCount = MBB->size();
    for(MachineBasicBlock::iterator RegionEnd = MBB->end();
        RegionEnd != MBB->begin();) {
      // The next region starts above the previous region. Look backward in the
      // instruction stream until we find the nearest boundary.
      MachineBasicBlock::iterator I = RegionEnd;
      for(;I != MBB->begin(); --I, --RemainingCount) {
        if (TII->isSchedulingBoundary(*std::prev(I), &*MBB, Fn))
          break;
      }
      I = MBB->begin();

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `llvm::make_early_inc_range`, `std::prev`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`llvm::make_early_inc_range`, `std::prev`。

### Lines 371-402: Registers LLVM passes
```cpp
      // Skip empty scheduling regions.
      if (I == RegionEnd) {
        RegionEnd = std::prev(RegionEnd);
        --RemainingCount;
        continue;
      }
      // Skip regions with one instruction.
      if (I == std::prev(RegionEnd)) {
        RegionEnd = std::prev(RegionEnd);
        continue;
      }

      Packetizer.PacketizeMIs(&*MBB, &*I, RegionEnd);
      RegionEnd = I;
    }
  }

  return true;

}

} // end anonymous namespace

INITIALIZE_PASS_BEGIN(R600Packetizer, DEBUG_TYPE,
                     "R600 Packetizer", false, false)
INITIALIZE_PASS_END(R600Packetizer, DEBUG_TYPE,
                    "R600 Packetizer", false, false)

char R600Packetizer::ID = 0;

char &llvm::R600PacketizerID = R600Packetizer::ID;

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `std::prev`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`std::prev`。

### Lines 403-405: Implements llvm::createR600Packetizer
```cpp
llvm::FunctionPass *llvm::createR600Packetizer() {
  return new R600Packetizer();
}
```
**EN:** This section contains concrete logic for llvm::createR600Packetizer. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `llvm::createR600Packetizer`.
**CN:** 本节包含与 llvm::createR600Packetizer 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`llvm::createR600Packetizer`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `R600Packetizer`, `R600PacketizerList`, `MachineFunctionPass::getAnalysisUsage`, `VLIWPacketizerList::addToPacket`, `std::next`, `R600Packetizer::runOnMachineFunction`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; scheduling / 调度
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"MCTargetDesc/R600MCTargetDesc.h"`
- `"R600.h"`
- `"R600Subtarget.h"`
- `"llvm/CodeGen/DFAPacketizer.h"`
- `"llvm/CodeGen/MachineDominators.h"`
- `"llvm/CodeGen/MachineLoopInfo.h"`
- `"llvm/CodeGen/ScheduleDAG.h"`
