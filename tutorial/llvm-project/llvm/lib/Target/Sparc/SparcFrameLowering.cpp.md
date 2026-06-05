# SparcFrameLowering.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/SparcFrameLowering.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements stack frame layout plus prologue/epilogue emission for the backend.
  - **CN**: 实现该后端的栈帧布局以及序言/尾声生成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- SparcFrameLowering.cpp - Sparc Frame Information ------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the Sparc implementation of TargetFrameLowering class.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 12-25
```cpp

#include "SparcFrameLowering.h"
#include "SparcInstrInfo.h"
#include "SparcMachineFunctionInfo.h"
#include "SparcSubtarget.h"
#include "llvm/CodeGen/CFIInstBuilder.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineModuleInfo.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Target/TargetOptions.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `SparcFrameLowering.h`, `SparcInstrInfo.h`, `SparcMachineFunctionInfo.h`, `SparcSubtarget.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `SparcFrameLowering.h`, `SparcInstrInfo.h`, `SparcMachineFunctionInfo.h`, `SparcSubtarget.h`。

### Lines 26-38
```cpp
using namespace llvm;

static cl::opt<bool>
DisableLeafProc("disable-sparc-leaf-proc",
                cl::init(false),
                cl::desc("Disable Sparc leaf procedure optimization."),
                cl::Hidden);

SparcFrameLowering::SparcFrameLowering(const SparcSubtarget &ST)
    : TargetFrameLowering(TargetFrameLowering::StackGrowsDown,
                          ST.is64Bit() ? Align(16) : Align(8), 0,
                          ST.is64Bit() ? Align(16) : Align(8),
                          /*StackRealignable=*/false) {}
```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 39-50
```cpp

void SparcFrameLowering::emitSPAdjustment(MachineFunction &MF,
                                          MachineBasicBlock &MBB,
                                          MachineBasicBlock::iterator MBBI,
                                          int NumBytes,
                                          unsigned ADDrr,
                                          unsigned ADDri) const {

  DebugLoc dl;
  const SparcInstrInfo &TII =
      *static_cast<const SparcInstrInfo *>(MF.getSubtarget().getInstrInfo());

```
- **EN**: Implements logic around `emitSPAdjustment`, `getSubtarget`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `emitSPAdjustment`, `getSubtarget` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 51-68
```cpp
  if (NumBytes >= -4096 && NumBytes < 4096) {
    BuildMI(MBB, MBBI, dl, TII.get(ADDri), SP::O6)
      .addReg(SP::O6).addImm(NumBytes);
    return;
  }

  // Emit this the hard way.  This clobbers G1 which we always know is
  // available here.
  if (NumBytes >= 0) {
    // Emit nonnegative numbers with sethi + or.
    // sethi %hi(NumBytes), %g1
    // or %g1, %lo(NumBytes), %g1
    // add %sp, %g1, %sp
    BuildMI(MBB, MBBI, dl, TII.get(SP::SETHIi), SP::G1)
      .addImm(HI22(NumBytes));
    BuildMI(MBB, MBBI, dl, TII.get(SP::ORri), SP::G1)
      .addReg(SP::G1).addImm(LO10(NumBytes));
    BuildMI(MBB, MBBI, dl, TII.get(ADDrr), SP::O6)
```
- **EN**: Implements logic around `BuildMI`, `addReg`, `addImm`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addReg`, `addImm` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 69-84
```cpp
      .addReg(SP::O6).addReg(SP::G1);
    return ;
  }

  // Emit negative numbers with sethi + xor.
  // sethi %hix(NumBytes), %g1
  // xor %g1, %lox(NumBytes), %g1
  // add %sp, %g1, %sp
  BuildMI(MBB, MBBI, dl, TII.get(SP::SETHIi), SP::G1)
    .addImm(HIX22(NumBytes));
  BuildMI(MBB, MBBI, dl, TII.get(SP::XORri), SP::G1)
    .addReg(SP::G1).addImm(LOX10(NumBytes));
  BuildMI(MBB, MBBI, dl, TII.get(ADDrr), SP::O6)
    .addReg(SP::O6).addReg(SP::G1);
}

```
- **EN**: Implements logic around `addReg`, `BuildMI`, `addImm`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `addReg`, `BuildMI`, `addImm` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 85-93
```cpp
void SparcFrameLowering::emitPrologue(MachineFunction &MF,
                                      MachineBasicBlock &MBB) const {
  SparcMachineFunctionInfo *FuncInfo = MF.getInfo<SparcMachineFunctionInfo>();

  assert(&MF.front() == &MBB && "Shrink-wrapping not yet supported");
  MachineFrameInfo &MFI = MF.getFrameInfo();
  const SparcSubtarget &Subtarget = MF.getSubtarget<SparcSubtarget>();
  MachineBasicBlock::iterator MBBI = MBB.begin();

```
- **EN**: Implements logic around `emitPrologue`, `getInfo<SparcMachineFunctionInfo>`, `assert`, `getFrameInfo`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `emitPrologue`, `getInfo<SparcMachineFunctionInfo>`, `assert`, `getFrameInfo`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 94-105
```cpp
  // Get the number of bytes to allocate from the FrameInfo
  int NumBytes = (int) MFI.getStackSize();

  unsigned SAVEri = SP::SAVEri;
  unsigned SAVErr = SP::SAVErr;
  if (FuncInfo->isLeafProc()) {
    if (NumBytes == 0)
      return;
    SAVEri = SP::ADDri;
    SAVErr = SP::ADDrr;
  }

```
- **EN**: Implements logic around `getStackSize`; this block applies conditional target rules.
- **CN**: 围绕 `getStackSize` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 106-117
```cpp
  // The SPARC ABI is a bit odd in that it requires a reserved 92-byte
  // (128 in v9) area in the user's stack, starting at %sp. Thus, the
  // first part of the stack that can actually be used is located at
  // %sp + 92.
  //
  // We therefore need to add that offset to the total stack size
  // after all the stack objects are placed by
  // PrologEpilogInserter calculateFrameObjectOffsets. However, since the stack needs to be
  // aligned *after* the extra size is added, we need to disable
  // calculateFrameObjectOffsets's built-in stack alignment, by having
  // targetHandlesStackFrameRounding return true.

```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 118-128
```cpp

  // Add the extra call frame stack size, if needed. (This is the same
  // code as in PrologEpilogInserter, but also gets disabled by
  // targetHandlesStackFrameRounding)
  if (MFI.adjustsStack() && hasReservedCallFrame(MF))
    NumBytes += MFI.getMaxCallFrameSize();

  // Adds the SPARC subtarget-specific spill area to the stack
  // size. Also ensures target-required alignment.
  NumBytes = Subtarget.getAdjustedFrameSize(NumBytes);

```
- **EN**: Implements logic around `getMaxCallFrameSize`, `getAdjustedFrameSize`; this block applies conditional target rules.
- **CN**: 围绕 `getMaxCallFrameSize`, `getAdjustedFrameSize` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 129-137
```cpp
  // Finally, ensure that the size is sufficiently aligned for the
  // data on the stack.
  NumBytes = alignTo(NumBytes, MFI.getMaxAlign());

  // Update stack size with corrected value.
  MFI.setStackSize(NumBytes);

  emitSPAdjustment(MF, MBB, MBBI, -NumBytes, SAVErr, SAVEri);

```
- **EN**: Implements logic around `alignTo`, `setStackSize`, `emitSPAdjustment`.
- **CN**: 围绕 `alignTo`, `setStackSize`, `emitSPAdjustment` 实现具体逻辑。

### Lines 138-154
```cpp
  if (MF.needsFrameMoves()) {
    CFIInstBuilder CFIBuilder(MBB, MBBI, MachineInstr::NoFlags);
    CFIBuilder.buildDefCFARegister(SP::I6);
    CFIBuilder.buildWindowSave();
    CFIBuilder.buildRegister(SP::O7, SP::I7);
  }
}

MachineBasicBlock::iterator SparcFrameLowering::
eliminateCallFramePseudoInstr(MachineFunction &MF, MachineBasicBlock &MBB,
                              MachineBasicBlock::iterator I) const {
  if (!hasReservedCallFrame(MF)) {
    MachineInstr &MI = *I;
    int Size = MI.getOperand(0).getImm();
    if (MI.getOpcode() == SP::ADJCALLSTACKDOWN)
      Size = -Size;

```
- **EN**: Implements logic around `CFIBuilder`, `buildDefCFARegister`, `buildWindowSave`, `buildRegister`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `CFIBuilder`, `buildDefCFARegister`, `buildWindowSave`, `buildRegister`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 155-172
```cpp
    if (Size)
      emitSPAdjustment(MF, MBB, I, Size, SP::ADDrr, SP::ADDri);
  }
  return MBB.erase(I);
}


void SparcFrameLowering::emitEpilogue(MachineFunction &MF,
                                  MachineBasicBlock &MBB) const {
  SparcMachineFunctionInfo *FuncInfo = MF.getInfo<SparcMachineFunctionInfo>();
  MachineBasicBlock::iterator MBBI = MBB.getLastNonDebugInstr();
  const SparcInstrInfo &TII =
      *static_cast<const SparcInstrInfo *>(MF.getSubtarget().getInstrInfo());
  DebugLoc dl = MBBI->getDebugLoc();
  assert((MBBI->getOpcode() == SP::RETL || MBBI->getOpcode() == SP::TAIL_CALL ||
          MBBI->getOpcode() == SP::TAIL_CALLri) &&
         "Can only put epilog before 'retl' or 'tail_call' instruction!");
  if (!FuncInfo->isLeafProc()) {
```
- **EN**: Implements logic around `emitSPAdjustment`, `erase`, `emitEpilogue`, `getInfo<SparcMachineFunctionInfo>`, ...; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `emitSPAdjustment`, `erase`, `emitEpilogue`, `getInfo<SparcMachineFunctionInfo>`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 173-182
```cpp
    BuildMI(MBB, MBBI, dl, TII.get(SP::RESTORErr), SP::G0).addReg(SP::G0)
      .addReg(SP::G0);
    return;
  }
  MachineFrameInfo &MFI = MF.getFrameInfo();

  int NumBytes = (int) MFI.getStackSize();
  if (NumBytes != 0)
    emitSPAdjustment(MF, MBB, MBBI, NumBytes, SP::ADDrr, SP::ADDri);

```
- **EN**: Implements logic around `BuildMI`, `addReg`, `getFrameInfo`, `getStackSize`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addReg`, `getFrameInfo`, `getStackSize`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 183-194
```cpp
  // Preserve return address in %o7
  if (MBBI->getOpcode() == SP::TAIL_CALL) {
    MBB.addLiveIn(SP::O7);
    BuildMI(MBB, MBBI, dl, TII.get(SP::ORrr), SP::G1)
        .addReg(SP::G0)
        .addReg(SP::O7);
    BuildMI(MBB, MBBI, dl, TII.get(SP::ORrr), SP::O7)
        .addReg(SP::G0)
        .addReg(SP::G1);
  }
}

```
- **EN**: Implements logic around `addLiveIn`, `BuildMI`, `addReg`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `addLiveIn`, `BuildMI`, `addReg` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 195-208
```cpp
bool SparcFrameLowering::hasReservedCallFrame(const MachineFunction &MF) const {
  // Reserve call frame if there are no variable sized objects on the stack.
  return !MF.getFrameInfo().hasVarSizedObjects();
}

// hasFPImpl - Return true if the specified function should have a dedicated
// frame pointer register.  This is true if the function has variable sized
// allocas or if frame pointer elimination is disabled.
bool SparcFrameLowering::hasFPImpl(const MachineFunction &MF) const {
  const MachineFrameInfo &MFI = MF.getFrameInfo();
  return MF.getTarget().Options.DisableFramePointerElim(MF) ||
         MFI.hasVarSizedObjects() || MFI.isFrameAddressTaken();
}

```
- **EN**: Implements logic around `hasReservedCallFrame`, `getFrameInfo`, `hasFPImpl`, `getTarget`, ...; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `hasReservedCallFrame`, `getFrameInfo`, `hasFPImpl`, `getTarget`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 209-217
```cpp
StackOffset
SparcFrameLowering::getFrameIndexReference(const MachineFunction &MF, int FI,
                                           Register &FrameReg) const {
  const SparcSubtarget &Subtarget = MF.getSubtarget<SparcSubtarget>();
  const MachineFrameInfo &MFI = MF.getFrameInfo();
  const SparcRegisterInfo *RegInfo = Subtarget.getRegisterInfo();
  const SparcMachineFunctionInfo *FuncInfo = MF.getInfo<SparcMachineFunctionInfo>();
  bool isFixed = MFI.isFixedObjectIndex(FI);

```
- **EN**: Implements logic around `getFrameIndexReference`, `getSubtarget<SparcSubtarget>`, `getFrameInfo`, `getRegisterInfo`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getFrameIndexReference`, `getSubtarget<SparcSubtarget>`, `getFrameInfo`, `getRegisterInfo`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 218-235
```cpp
  // Addressable stack objects are accessed using neg. offsets from
  // %fp, or positive offsets from %sp.
  bool UseFP;

  // Sparc uses FP-based references in general, even when "hasFP" is
  // false. That function is rather a misnomer, because %fp is
  // actually always available, unless isLeafProc.
  if (FuncInfo->isLeafProc()) {
    // If there's a leaf proc, all offsets need to be %sp-based,
    // because we haven't caused %fp to actually point to our frame.
    UseFP = false;
  } else if (isFixed) {
    // Otherwise, argument access should always use %fp.
    UseFP = true;
  } else {
    // Finally, default to using %fp.
    UseFP = true;
  }
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 236-248
```cpp

  int64_t FrameOffset = MF.getFrameInfo().getObjectOffset(FI) +
      Subtarget.getStackPointerBias();

  if (UseFP) {
    FrameReg = RegInfo->getFrameRegister(MF);
    return StackOffset::getFixed(FrameOffset);
  } else {
    FrameReg = SP::O6; // %sp
    return StackOffset::getFixed(FrameOffset + MF.getFrameInfo().getStackSize());
  }
}

```
- **EN**: Implements logic around `getFrameInfo`, `getStackPointerBias`, `getFrameRegister`, `getFixed`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getFrameInfo`, `getStackPointerBias`, `getFrameRegister`, `getFixed` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 249-258
```cpp
[[maybe_unused]] static bool verifyLeafProcRegUse(MachineRegisterInfo *MRI) {

  for (unsigned reg = SP::I0; reg <= SP::I7; ++reg)
    if (MRI->isPhysRegUsed(reg))
      return false;

  for (unsigned reg = SP::L0; reg <= SP::L7; ++reg)
    if (MRI->isPhysRegUsed(reg))
      return false;

```
- **EN**: Implements logic around `verifyLeafProcRegUse`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `verifyLeafProcRegUse` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 259-267
```cpp
  return true;
}

bool SparcFrameLowering::isLeafProc(MachineFunction &MF) const
{

  MachineRegisterInfo &MRI = MF.getRegInfo();
  MachineFrameInfo    &MFI = MF.getFrameInfo();

```
- **EN**: Implements logic around `isLeafProc`, `getRegInfo`, `getFrameInfo`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `isLeafProc`, `getRegInfo`, `getFrameInfo` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 268-281
```cpp
  return !(MFI.hasCalls()               // has calls
           || MRI.isPhysRegUsed(SP::L0) // Too many registers needed
           || MRI.isPhysRegUsed(SP::O6) // %sp is used
           || hasFP(MF)                 // need %fp
           || MF.hasInlineAsm());       // has inline assembly
}

void SparcFrameLowering::remapRegsForLeafProc(MachineFunction &MF) const {
  MachineRegisterInfo &MRI = MF.getRegInfo();
  // Remap %i[0-7] to %o[0-7].
  for (unsigned reg = SP::I0; reg <= SP::I7; ++reg) {
    if (!MRI.isPhysRegUsed(reg))
      continue;

```
- **EN**: Implements logic around `hasCalls`, `isPhysRegUsed`, `hasFP`, `hasInlineAsm`, ...; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `hasCalls`, `isPhysRegUsed`, `hasFP`, `hasInlineAsm`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 282-294
```cpp
    unsigned mapped_reg = reg - SP::I0 + SP::O0;

    // Replace I register with O register.
    MRI.replaceRegWith(reg, mapped_reg);

    // Also replace register pair super-registers.
    if ((reg - SP::I0) % 2 == 0) {
      unsigned preg = (reg - SP::I0) / 2 + SP::I0_I1;
      unsigned mapped_preg = preg - SP::I0_I1 + SP::O0_O1;
      MRI.replaceRegWith(preg, mapped_preg);
    }
  }

```
- **EN**: Implements logic around `replaceRegWith`; this block applies conditional target rules.
- **CN**: 围绕 `replaceRegWith` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 295-310
```cpp
  // Rewrite MBB's Live-ins.
  for (MachineBasicBlock &MBB : MF) {
    for (unsigned reg = SP::I0_I1; reg <= SP::I6_I7; ++reg) {
      if (!MBB.isLiveIn(reg))
        continue;
      MBB.removeLiveIn(reg);
      MBB.addLiveIn(reg - SP::I0_I1 + SP::O0_O1);
    }
    for (unsigned reg = SP::I0; reg <= SP::I7; ++reg) {
      if (!MBB.isLiveIn(reg))
        continue;
      MBB.removeLiveIn(reg);
      MBB.addLiveIn(reg - SP::I0 + SP::O0);
    }
  }

```
- **EN**: Implements logic around `removeLiveIn`, `addLiveIn`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `removeLiveIn`, `addLiveIn` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 311-324
```cpp
  assert(verifyLeafProcRegUse(&MRI));
#ifdef EXPENSIVE_CHECKS
  MF.verify(0, "After LeafProc Remapping");
#endif
}

void SparcFrameLowering::determineCalleeSaves(MachineFunction &MF,
                                              BitVector &SavedRegs,
                                              RegScavenger *RS) const {
  TargetFrameLowering::determineCalleeSaves(MF, SavedRegs, RS);
  if (!DisableLeafProc && isLeafProc(MF)) {
    SparcMachineFunctionInfo *MFI = MF.getInfo<SparcMachineFunctionInfo>();
    MFI->setLeafProc(true);

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 325-328
```cpp
    remapRegsForLeafProc(MF);
  }

}
```
- **EN**: Implements logic around `remapRegsForLeafProc`.
- **CN**: 围绕 `remapRegsForLeafProc` 实现具体逻辑。

## Key Concepts / 关键概念

- **Stack frame management / 栈帧管理**:
  - **EN**: Controls prologue/epilogue emission and frame layout
  - **CN**: 控制序言尾声生成与栈帧布局
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `SparcFrameLowering.h`, `SparcInstrInfo.h`, `SparcMachineFunctionInfo.h`, `SparcSubtarget.h`, `llvm/CodeGen/CFIInstBuilder.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineModuleInfo.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/Support/CommandLine.h`, `llvm/Target/TargetOptions.h`
- **LLVM subsystems / LLVM 子系统**: CodeGen, Support
