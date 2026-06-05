# VERegisterInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/VE/VERegisterInfo.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements target-specific register information, reserved registers, spills, and frame-index handling.
  - **CN**: 实现目标相关的寄存器信息、保留寄存器、溢出处理以及 frame index 处理。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- VERegisterInfo.cpp - VE Register Information ----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the VE implementation of the TargetRegisterInfo class.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 12-23
```cpp

#include "VERegisterInfo.h"
#include "VE.h"
#include "VESubtarget.h"
#include "llvm/ADT/BitVector.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
#include "llvm/IR/Type.h"
#include "llvm/Support/Debug.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `VERegisterInfo.h`, `VE.h`, `VESubtarget.h`, `llvm/ADT/BitVector.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `VERegisterInfo.h`, `VE.h`, `VESubtarget.h`, `llvm/ADT/BitVector.h`。

### Lines 24-33
```cpp
using namespace llvm;

#define DEBUG_TYPE "ve-register-info"

#define GET_REGINFO_TARGET_DESC
#include "VEGenRegisterInfo.inc"

// VE uses %s10 == %lp to keep return address
VERegisterInfo::VERegisterInfo() : VEGenRegisterInfo(VE::SX10) {}

```
- **EN**: Pulls in the headers needed for this implementation, including `VEGenRegisterInfo.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `VEGenRegisterInfo.inc`。

### Lines 34-45
```cpp
const MCPhysReg *
VERegisterInfo::getCalleeSavedRegs(const MachineFunction *MF) const {
  switch (MF->getFunction().getCallingConv()) {
  case CallingConv::Fast:
    // Being explicit (same as standard CC).
  default:
    return CSR_SaveList;
  case CallingConv::PreserveAll:
    return CSR_preserve_all_SaveList;
  }
}

```
- **EN**: Implements logic around `getCalleeSavedRegs`; this block uses `switch`-based dispatch; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getCalleeSavedRegs` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 46-57
```cpp
const uint32_t *VERegisterInfo::getCallPreservedMask(const MachineFunction &MF,
                                                     CallingConv::ID CC) const {
  switch (CC) {
  case CallingConv::Fast:
    // Being explicit (same as standard CC).
  default:
    return CSR_RegMask;
  case CallingConv::PreserveAll:
    return CSR_preserve_all_RegMask;
  }
}

```
- **EN**: Implements logic around `getCallPreservedMask`; this block uses `switch`-based dispatch; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getCallPreservedMask` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 58-70
```cpp
const uint32_t *VERegisterInfo::getNoPreservedMask() const {
  return CSR_NoRegs_RegMask;
}

BitVector VERegisterInfo::getReservedRegs(const MachineFunction &MF) const {
  BitVector Reserved(getNumRegs());

  const Register ReservedRegs[] = {
      VE::SX8,  // Stack limit
      VE::SX9,  // Frame pointer
      VE::SX10, // Link register (return address)
      VE::SX11, // Stack pointer

```
- **EN**: Implements logic around `getNoPreservedMask`, `getReservedRegs`, `Reserved`, `register`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getNoPreservedMask`, `getReservedRegs`, `Reserved`, `register` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 71-82
```cpp
      // FIXME: maybe not need to be reserved
      VE::SX12, // Outer register
      VE::SX13, // Id register for dynamic linker

      VE::SX14, // Thread pointer
      VE::SX15, // Global offset table register
      VE::SX16, // Procedure linkage table register
      VE::SX17, // Linkage-area register
                // sx18-sx33 are callee-saved registers
                // sx34-sx63 are temporary registers
  };

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 83-91
```cpp
  for (auto R : ReservedRegs)
    for (MCRegAliasIterator ItAlias(R, this, true); ItAlias.isValid();
         ++ItAlias)
      Reserved.set(*ItAlias);

  // Reserve constant registers.
  Reserved.set(VE::VM0);
  Reserved.set(VE::VMP0);

```
- **EN**: Implements logic around `set`.
- **CN**: 围绕 `set` 实现具体逻辑。

### Lines 92-103
```cpp
  return Reserved;
}

const TargetRegisterClass *
VERegisterInfo::getPointerRegClass(unsigned Kind) const {
  return &VE::I64RegClass;
}

static unsigned offsetToDisp(MachineInstr &MI) {
  // Default offset in instruction's operands (reg+reg+imm).
  unsigned OffDisp = 2;

```
- **EN**: Implements logic around `getPointerRegClass`, `offsetToDisp`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getPointerRegClass`, `offsetToDisp` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 104-120
```cpp
#define RRCAS_multi_cases(NAME) NAME##rir : case NAME##rii

  {
    using namespace llvm::VE;
    switch (MI.getOpcode()) {
    case INLINEASM:
    case RRCAS_multi_cases(TS1AML):
    case RRCAS_multi_cases(TS1AMW):
    case RRCAS_multi_cases(CASL):
    case RRCAS_multi_cases(CASW):
      // These instructions use AS format (reg+imm).
      OffDisp = 1;
      break;
    }
  }
#undef RRCAS_multi_cases

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 121-132
```cpp
  return OffDisp;
}

namespace {
class EliminateFrameIndex {
  const TargetInstrInfo &TII;
  const TargetRegisterInfo &TRI;
  const DebugLoc &DL;
  MachineBasicBlock &MBB;
  MachineBasicBlock::iterator II;
  Register clobber;

```
- **EN**: Introduces declarations for `EliminateFrameIndex`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `EliminateFrameIndex` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 133-150
```cpp
  // Some helper functions for the ease of instruction building.
  MachineFunction &getFunc() const { return *MBB.getParent(); }
  inline MCRegister getSubReg(MCRegister Reg, unsigned Idx) const {
    return TRI.getSubReg(Reg, Idx);
  }
  inline const MCInstrDesc &get(unsigned Opcode) const {
    return TII.get(Opcode);
  }
  inline MachineInstrBuilder build(const MCInstrDesc &MCID, Register DestReg) {
    return BuildMI(MBB, II, DL, MCID, DestReg);
  }
  inline MachineInstrBuilder build(unsigned InstOpc, Register DestReg) {
    return build(get(InstOpc), DestReg);
  }
  inline MachineInstrBuilder build(const MCInstrDesc &MCID) {
    return BuildMI(MBB, II, DL, MCID);
  }
  inline MachineInstrBuilder build(unsigned InstOpc) {
```
- **EN**: Implements logic around `getFunc`, `getSubReg`, `get`, `build`, ...; this block returns target-specific results; works at the MachineInstr/MachineFunction layer; works at the MC layer.
- **CN**: 围绕 `getFunc`, `getSubReg`, `get`, `build`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层，工作在 MC 层。

### Lines 151-164
```cpp
    return build(get(InstOpc));
  }

  // Calculate an address of frame index from a frame register and a given
  // offset if the offset doesn't fit in the immediate field.  Use a clobber
  // register to hold calculated address.
  void prepareReplaceFI(MachineInstr &MI, Register &FrameReg, int64_t &Offset,
                        int64_t Bytes = 0);
  // Replace the frame index in \p MI with a frame register and a given offset
  // if it fits in the immediate field.  Otherwise, use pre-calculated address
  // in a clobber regsiter.
  void replaceFI(MachineInstr &MI, Register FrameReg, int64_t Offset,
                 int FIOperandNum);

```
- **EN**: Implements logic around `build`, `prepareReplaceFI`, `replaceFI`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `build`, `prepareReplaceFI`, `replaceFI` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 165-180
```cpp
  // Expand and eliminate Frame Index of pseudo STQrii and LDQrii.
  void processSTQ(MachineInstr &MI, Register FrameReg, int64_t Offset,
                  int FIOperandNum);
  void processLDQ(MachineInstr &MI, Register FrameReg, int64_t Offset,
                  int FIOperandNum);
  // Expand and eliminate Frame Index of pseudo STVMrii and LDVMrii.
  void processSTVM(MachineInstr &MI, Register FrameReg, int64_t Offset,
                   int FIOperandNum);
  void processLDVM(MachineInstr &MI, Register FrameReg, int64_t Offset,
                   int FIOperandNum);
  // Expand and eliminate Frame Index of pseudo STVM512rii and LDVM512rii.
  void processSTVM512(MachineInstr &MI, Register FrameReg, int64_t Offset,
                      int FIOperandNum);
  void processLDVM512(MachineInstr &MI, Register FrameReg, int64_t Offset,
                      int FIOperandNum);

```
- **EN**: Implements logic around `processSTQ`, `processLDQ`, `processSTVM`, `processLDVM`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `processSTQ`, `processLDQ`, `processSTVM`, `processLDVM`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 181-192
```cpp
public:
  EliminateFrameIndex(const TargetInstrInfo &TII, const TargetRegisterInfo &TRI,
                      const DebugLoc &DL, MachineBasicBlock &MBB,
                      MachineBasicBlock::iterator II)
      : TII(TII), TRI(TRI), DL(DL), MBB(MBB), II(II), clobber(VE::SX13) {}

  // Expand and eliminate Frame Index from MI
  void processMI(MachineInstr &MI, Register FrameReg, int64_t Offset,
                 int FIOperandNum);
};
} // namespace

```
- **EN**: Implements logic around `EliminateFrameIndex`, `TII`, `processMI`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `EliminateFrameIndex`, `TII`, `processMI` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 193-202
```cpp
// Prepare the frame index if it doesn't fit in the immediate field.  Use
// clobber register to hold calculated address.
void EliminateFrameIndex::prepareReplaceFI(MachineInstr &MI, Register &FrameReg,
                                           int64_t &Offset, int64_t Bytes) {
  if (isInt<32>(Offset) && isInt<32>(Offset + Bytes)) {
    // If the offset is small enough to fit in the immediate field, directly
    // encode it.  So, nothing to prepare here.
    return;
  }

```
- **EN**: Implements logic around `prepareReplaceFI`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `prepareReplaceFI` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 203-214
```cpp
  // If the offset doesn't fit, emit following codes.  This clobbers SX13
  // which we always know is available here.
  //   lea     %clobber, Offset@lo
  //   and     %clobber, %clobber, (32)0
  //   lea.sl  %clobber, Offset@hi(FrameReg, %clobber)
  build(VE::LEAzii, clobber).addImm(0).addImm(0).addImm(Lo_32(Offset));
  build(VE::ANDrm, clobber).addReg(clobber).addImm(M0(32));
  build(VE::LEASLrri, clobber)
      .addReg(clobber)
      .addReg(FrameReg)
      .addImm(Hi_32(Offset));

```
- **EN**: Implements logic around `build`, `addReg`, `addImm`.
- **CN**: 围绕 `build`, `addReg`, `addImm` 实现具体逻辑。

### Lines 215-224
```cpp
  // Use clobber register as a frame register and 0 offset
  FrameReg = clobber;
  Offset = 0;
}

// Replace the frame index in \p MI with a proper byte and framereg offset.
void EliminateFrameIndex::replaceFI(MachineInstr &MI, Register FrameReg,
                                    int64_t Offset, int FIOperandNum) {
  assert(isInt<32>(Offset));

```
- **EN**: Implements logic around `replaceFI`, `assert`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `replaceFI`, `assert` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 225-235
```cpp
  // The offset must be small enough to fit in the immediate field after
  // call of prepareReplaceFI.  Therefore, we directly encode it.
  MI.getOperand(FIOperandNum).ChangeToRegister(FrameReg, false);
  MI.getOperand(FIOperandNum + offsetToDisp(MI)).ChangeToImmediate(Offset);
}

void EliminateFrameIndex::processSTQ(MachineInstr &MI, Register FrameReg,
                                     int64_t Offset, int FIOperandNum) {
  assert(MI.getOpcode() == VE::STQrii);
  LLVM_DEBUG(dbgs() << "processSTQ: "; MI.dump());

```
- **EN**: Implements logic around `getOperand`, `processSTQ`, `assert`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOperand`, `processSTQ`, `assert` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 236-251
```cpp
  prepareReplaceFI(MI, FrameReg, Offset, 8);

  Register SrcReg = MI.getOperand(3).getReg();
  Register SrcHiReg = getSubReg(SrcReg, VE::sub_even);
  Register SrcLoReg = getSubReg(SrcReg, VE::sub_odd);
  // VE stores HiReg to 8(addr) and LoReg to 0(addr)
  MachineInstr *StMI =
      build(VE::STrii).addReg(FrameReg).addImm(0).addImm(0).addReg(SrcLoReg);
  replaceFI(*StMI, FrameReg, Offset, 0);
  // Mutate to 'hi' store.
  MI.setDesc(get(VE::STrii));
  MI.getOperand(3).setReg(SrcHiReg);
  Offset += 8;
  replaceFI(MI, FrameReg, Offset, FIOperandNum);
}

```
- **EN**: Implements logic around `prepareReplaceFI`, `getOperand`, `getSubReg`, `build`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `prepareReplaceFI`, `getOperand`, `getSubReg`, `build`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 252-269
```cpp
void EliminateFrameIndex::processLDQ(MachineInstr &MI, Register FrameReg,
                                     int64_t Offset, int FIOperandNum) {
  assert(MI.getOpcode() == VE::LDQrii);
  LLVM_DEBUG(dbgs() << "processLDQ: "; MI.dump());

  prepareReplaceFI(MI, FrameReg, Offset, 8);

  Register DestReg = MI.getOperand(0).getReg();
  Register DestHiReg = getSubReg(DestReg, VE::sub_even);
  Register DestLoReg = getSubReg(DestReg, VE::sub_odd);
  // VE loads HiReg from 8(addr) and LoReg from 0(addr)
  MachineInstr *StMI =
      build(VE::LDrii, DestLoReg).addReg(FrameReg).addImm(0).addImm(0);
  replaceFI(*StMI, FrameReg, Offset, 1);
  MI.setDesc(get(VE::LDrii));
  MI.getOperand(0).setReg(DestHiReg);
  Offset += 8;
  replaceFI(MI, FrameReg, Offset, FIOperandNum);
```
- **EN**: Implements logic around `processLDQ`, `assert`, `prepareReplaceFI`, `getOperand`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `processLDQ`, `assert`, `prepareReplaceFI`, `getOperand`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 270-287
```cpp
}

void EliminateFrameIndex::processSTVM(MachineInstr &MI, Register FrameReg,
                                      int64_t Offset, int FIOperandNum) {
  assert(MI.getOpcode() == VE::STVMrii);
  LLVM_DEBUG(dbgs() << "processSTVM: "; MI.dump());

  // Original MI is:
  //   STVMrii frame-index, 0, offset, reg (, memory operand)
  // Convert it to:
  //   SVMi   tmp-reg, reg, 0
  //   STrii  frame-reg, 0, offset, tmp-reg
  //   SVMi   tmp-reg, reg, 1
  //   STrii  frame-reg, 0, offset+8, tmp-reg
  //   SVMi   tmp-reg, reg, 2
  //   STrii  frame-reg, 0, offset+16, tmp-reg
  //   SVMi   tmp-reg, reg, 3
  //   STrii  frame-reg, 0, offset+24, tmp-reg
```
- **EN**: Implements logic around `processSTVM`, `assert`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `processSTVM`, `assert` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 288-305
```cpp

  prepareReplaceFI(MI, FrameReg, Offset, 24);

  Register SrcReg = MI.getOperand(3).getReg();
  bool isKill = MI.getOperand(3).isKill();
  // FIXME: it would be better to scavenge a register here instead of
  // reserving SX16 all of the time.
  Register TmpReg = VE::SX16;
  for (int i = 0; i < 3; ++i) {
    build(VE::SVMmr, TmpReg).addReg(SrcReg).addImm(i);
    MachineInstr *StMI =
        build(VE::STrii).addReg(FrameReg).addImm(0).addImm(0).addReg(
            TmpReg, getKillRegState(true));
    replaceFI(*StMI, FrameReg, Offset, 0);
    Offset += 8;
  }
  build(VE::SVMmr, TmpReg).addReg(SrcReg, getKillRegState(isKill)).addImm(3);
  MI.setDesc(get(VE::STrii));
```
- **EN**: Implements logic around `prepareReplaceFI`, `getOperand`, `build`, `getKillRegState`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `prepareReplaceFI`, `getOperand`, `build`, `getKillRegState`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 306-314
```cpp
  MI.getOperand(3).ChangeToRegister(TmpReg, false, false, true);
  replaceFI(MI, FrameReg, Offset, FIOperandNum);
}

void EliminateFrameIndex::processLDVM(MachineInstr &MI, Register FrameReg,
                                      int64_t Offset, int FIOperandNum) {
  assert(MI.getOpcode() == VE::LDVMrii);
  LLVM_DEBUG(dbgs() << "processLDVM: "; MI.dump());

```
- **EN**: Implements logic around `getOperand`, `replaceFI`, `processLDVM`, `assert`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOperand`, `replaceFI`, `processLDVM`, `assert` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 315-326
```cpp
  // Original MI is:
  //   LDVMri reg, frame-index, 0, offset (, memory operand)
  // Convert it to:
  //   LDrii  tmp-reg, frame-reg, 0, offset
  //   LVMir vm, 0, tmp-reg
  //   LDrii  tmp-reg, frame-reg, 0, offset+8
  //   LVMir_m vm, 1, tmp-reg, vm
  //   LDrii  tmp-reg, frame-reg, 0, offset+16
  //   LVMir_m vm, 2, tmp-reg, vm
  //   LDrii  tmp-reg, frame-reg, 0, offset+24
  //   LVMir_m vm, 3, tmp-reg, vm

```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 327-344
```cpp
  prepareReplaceFI(MI, FrameReg, Offset, 24);

  Register DestReg = MI.getOperand(0).getReg();
  // FIXME: it would be better to scavenge a register here instead of
  // reserving SX16 all of the time.
  unsigned TmpReg = VE::SX16;
  for (int i = 0; i < 4; ++i) {
    if (i != 3) {
      MachineInstr *StMI =
          build(VE::LDrii, TmpReg).addReg(FrameReg).addImm(0).addImm(0);
      replaceFI(*StMI, FrameReg, Offset, 1);
      Offset += 8;
    } else {
      // Last LDrii replace the target instruction.
      MI.setDesc(get(VE::LDrii));
      MI.getOperand(0).ChangeToRegister(TmpReg, true);
    }
    // First LVM is LVMir.  Others are LVMir_m.  Last LVM places at the
```
- **EN**: Implements logic around `prepareReplaceFI`, `getOperand`, `build`, `replaceFI`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `prepareReplaceFI`, `getOperand`, `build`, `replaceFI`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 345-361
```cpp
    // next of the target instruction.
    if (i == 0)
      build(VE::LVMir, DestReg).addImm(i).addReg(TmpReg, getKillRegState(true));
    else if (i != 3)
      build(VE::LVMir_m, DestReg)
          .addImm(i)
          .addReg(TmpReg, getKillRegState(true))
          .addReg(DestReg);
    else
      BuildMI(*MI.getParent(), std::next(II), DL, get(VE::LVMir_m), DestReg)
          .addImm(3)
          .addReg(TmpReg, getKillRegState(true))
          .addReg(DestReg);
  }
  replaceFI(MI, FrameReg, Offset, FIOperandNum);
}

```
- **EN**: Implements logic around `build`, `addImm`, `addReg`, `BuildMI`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `build`, `addImm`, `addReg`, `BuildMI`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 362-379
```cpp
void EliminateFrameIndex::processSTVM512(MachineInstr &MI, Register FrameReg,
                                         int64_t Offset, int FIOperandNum) {
  assert(MI.getOpcode() == VE::STVM512rii);
  LLVM_DEBUG(dbgs() << "processSTVM512: "; MI.dump());

  prepareReplaceFI(MI, FrameReg, Offset, 56);

  Register SrcReg = MI.getOperand(3).getReg();
  Register SrcLoReg = getSubReg(SrcReg, VE::sub_vm_odd);
  Register SrcHiReg = getSubReg(SrcReg, VE::sub_vm_even);
  bool isKill = MI.getOperand(3).isKill();
  // FIXME: it would be better to scavenge a register here instead of
  // reserving SX16 all of the time.
  Register TmpReg = VE::SX16;
  // store low part of VMP
  MachineInstr *LastMI = nullptr;
  for (int i = 0; i < 4; ++i) {
    LastMI = build(VE::SVMmr, TmpReg).addReg(SrcLoReg).addImm(i);
```
- **EN**: Implements logic around `processSTVM512`, `assert`, `prepareReplaceFI`, `getOperand`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `processSTVM512`, `assert`, `prepareReplaceFI`, `getOperand`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 380-397
```cpp
    MachineInstr *StMI =
        build(VE::STrii).addReg(FrameReg).addImm(0).addImm(0).addReg(
            TmpReg, getKillRegState(true));
    replaceFI(*StMI, FrameReg, Offset, 0);
    Offset += 8;
  }
  if (isKill)
    LastMI->addRegisterKilled(SrcLoReg, &TRI, true);
  // store high part of VMP
  for (int i = 0; i < 3; ++i) {
    build(VE::SVMmr, TmpReg).addReg(SrcHiReg).addImm(i);
    MachineInstr *StMI =
        build(VE::STrii).addReg(FrameReg).addImm(0).addImm(0).addReg(
            TmpReg, getKillRegState(true));
    replaceFI(*StMI, FrameReg, Offset, 0);
    Offset += 8;
  }
  LastMI = build(VE::SVMmr, TmpReg).addReg(SrcHiReg).addImm(3);
```
- **EN**: Implements logic around `build`, `getKillRegState`, `replaceFI`, `addRegisterKilled`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `build`, `getKillRegState`, `replaceFI`, `addRegisterKilled` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 398-407
```cpp
  if (isKill) {
    LastMI->addRegisterKilled(SrcHiReg, &TRI, true);
    // Add implicit super-register kills to the particular MI.
    LastMI->addRegisterKilled(SrcReg, &TRI, true);
  }
  MI.setDesc(get(VE::STrii));
  MI.getOperand(3).ChangeToRegister(TmpReg, false, false, true);
  replaceFI(MI, FrameReg, Offset, FIOperandNum);
}

```
- **EN**: Implements logic around `addRegisterKilled`, `setDesc`, `getOperand`, `replaceFI`; this block applies conditional target rules.
- **CN**: 围绕 `addRegisterKilled`, `setDesc`, `getOperand`, `replaceFI` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 408-425
```cpp
void EliminateFrameIndex::processLDVM512(MachineInstr &MI, Register FrameReg,
                                         int64_t Offset, int FIOperandNum) {
  assert(MI.getOpcode() == VE::LDVM512rii);
  LLVM_DEBUG(dbgs() << "processLDVM512: "; MI.dump());

  prepareReplaceFI(MI, FrameReg, Offset, 56);

  Register DestReg = MI.getOperand(0).getReg();
  Register DestLoReg = getSubReg(DestReg, VE::sub_vm_odd);
  Register DestHiReg = getSubReg(DestReg, VE::sub_vm_even);
  // FIXME: it would be better to scavenge a register here instead of
  // reserving SX16 all of the time.
  Register TmpReg = VE::SX16;
  build(VE::IMPLICIT_DEF, DestReg);
  for (int i = 0; i < 4; ++i) {
    MachineInstr *LdMI =
        build(VE::LDrii, TmpReg).addReg(FrameReg).addImm(0).addImm(0);
    replaceFI(*LdMI, FrameReg, Offset, 1);
```
- **EN**: Implements logic around `processLDVM512`, `assert`, `prepareReplaceFI`, `getOperand`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `processLDVM512`, `assert`, `prepareReplaceFI`, `getOperand`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 426-443
```cpp
    build(VE::LVMir_m, DestLoReg)
        .addImm(i)
        .addReg(TmpReg, getKillRegState(true))
        .addReg(DestLoReg);
    Offset += 8;
  }
  for (int i = 0; i < 3; ++i) {
    MachineInstr *LdMI =
        build(VE::LDrii, TmpReg).addReg(FrameReg).addImm(0).addImm(0);
    replaceFI(*LdMI, FrameReg, Offset, 1);
    build(VE::LVMir_m, DestHiReg)
        .addImm(i)
        .addReg(TmpReg, getKillRegState(true))
        .addReg(DestHiReg);
    Offset += 8;
  }
  MI.setDesc(get(VE::LDrii));
  MI.getOperand(0).ChangeToRegister(TmpReg, true);
```
- **EN**: Implements logic around `build`, `addImm`, `addReg`, `replaceFI`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `build`, `addImm`, `addReg`, `replaceFI`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 444-461
```cpp
  BuildMI(*MI.getParent(), std::next(II), DL, get(VE::LVMir_m), DestHiReg)
      .addImm(3)
      .addReg(TmpReg, getKillRegState(true))
      .addReg(DestHiReg);
  replaceFI(MI, FrameReg, Offset, FIOperandNum);
}

void EliminateFrameIndex::processMI(MachineInstr &MI, Register FrameReg,
                                    int64_t Offset, int FIOperandNum) {
  switch (MI.getOpcode()) {
  case VE::STQrii:
    processSTQ(MI, FrameReg, Offset, FIOperandNum);
    return;
  case VE::LDQrii:
    processLDQ(MI, FrameReg, Offset, FIOperandNum);
    return;
  case VE::STVMrii:
    processSTVM(MI, FrameReg, Offset, FIOperandNum);
```
- **EN**: Implements logic around `BuildMI`, `addImm`, `addReg`, `replaceFI`, ...; this block uses `switch`-based dispatch; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addImm`, `addReg`, `replaceFI`, ... 实现具体逻辑；这一段使用 `switch` 分派，工作在 MachineInstr/MachineFunction 层。

### Lines 462-476
```cpp
    return;
  case VE::LDVMrii:
    processLDVM(MI, FrameReg, Offset, FIOperandNum);
    return;
  case VE::STVM512rii:
    processSTVM512(MI, FrameReg, Offset, FIOperandNum);
    return;
  case VE::LDVM512rii:
    processLDVM512(MI, FrameReg, Offset, FIOperandNum);
    return;
  }
  prepareReplaceFI(MI, FrameReg, Offset);
  replaceFI(MI, FrameReg, Offset, FIOperandNum);
}

```
- **EN**: Implements logic around `processLDVM`, `processSTVM512`, `processLDVM512`, `prepareReplaceFI`, ....
- **CN**: 围绕 `processLDVM`, `processSTVM512`, `processLDVM512`, `prepareReplaceFI`, ... 实现具体逻辑。

### Lines 477-492
```cpp
bool VERegisterInfo::eliminateFrameIndex(MachineBasicBlock::iterator II,
                                         int SPAdj, unsigned FIOperandNum,
                                         RegScavenger *RS) const {
  assert(SPAdj == 0 && "Unexpected");

  MachineInstr &MI = *II;
  int FrameIndex = MI.getOperand(FIOperandNum).getIndex();

  MachineFunction &MF = *MI.getParent()->getParent();
  const VESubtarget &Subtarget = MF.getSubtarget<VESubtarget>();
  const VEFrameLowering &TFI = *getFrameLowering(MF);
  const TargetInstrInfo &TII = *Subtarget.getInstrInfo();
  const VERegisterInfo &TRI = *Subtarget.getRegisterInfo();
  DebugLoc DL = MI.getDebugLoc();
  EliminateFrameIndex EFI(TII, TRI, DL, *MI.getParent(), II);

```
- **EN**: Implements logic around `eliminateFrameIndex`, `assert`, `getOperand`, `getParent`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `eliminateFrameIndex`, `assert`, `getOperand`, `getParent`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 493-502
```cpp
  // Retrieve FrameReg and byte offset for stack slot.
  Register FrameReg;
  int64_t Offset =
      TFI.getFrameIndexReference(MF, FrameIndex, FrameReg).getFixed();
  Offset += MI.getOperand(FIOperandNum + offsetToDisp(MI)).getImm();

  EFI.processMI(MI, FrameReg, Offset, FIOperandNum);
  return false;
}

```
- **EN**: Implements logic around `getFrameIndexReference`, `getOperand`, `processMI`; this block returns target-specific results.
- **CN**: 围绕 `getFrameIndexReference`, `getOperand`, `processMI` 实现具体逻辑；这一段返回目标相关结果。

### Lines 503-505
```cpp
Register VERegisterInfo::getFrameRegister(const MachineFunction &MF) const {
  return VE::SX9;
}
```
- **EN**: Implements logic around `getFrameRegister`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getFrameRegister` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

## Key Concepts / 关键概念

- **Register model / 寄存器模型**:
  - **EN**: Describes registers, classes, and allocation-facing details
  - **CN**: 描述寄存器、寄存器类以及面向分配器的细节
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `VERegisterInfo.h`, `VE.h`, `VESubtarget.h`, `llvm/ADT/BitVector.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/TargetInstrInfo.h`, `llvm/IR/Type.h`, `llvm/Support/Debug.h`, `VEGenRegisterInfo.inc`
- **LLVM subsystems / LLVM 子系统**: CodeGen, IR, Support
- **Generated macros / 生成宏**: `GET_REGINFO_TARGET_DESC`
