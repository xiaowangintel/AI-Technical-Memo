# MipsSEFrameLowering.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsSEFrameLowering.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains the Mips32/64 implementation of TargetFrameLowering class.
- 用途 (CN): 实现 Mips 后端中的 `MipsSEFrameLowering`，重点处理栈帧布局以及序言/结语生成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- MipsSEFrameLowering.cpp - Mips32/64 Frame Information --------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the Mips32/64 implementation of TargetFrameLowering class.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-27
```cpp
#include "MipsSEFrameLowering.h"
#include "MCTargetDesc/MipsABIInfo.h"
#include "MipsMachineFunction.h"
#include "MipsRegisterInfo.h"
#include "MipsSEInstrInfo.h"
#include "MipsSubtarget.h"
#include "llvm/ADT/BitVector.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/ADT/StringSwitch.h"
#include "llvm/CodeGen/CFIInstBuilder.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 28-42
```cpp
#include "llvm/CodeGen/MachineModuleInfo.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/RegisterScavenging.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include "llvm/IR/DebugLoc.h"
#include "llvm/IR/Function.h"
#include "llvm/Support/CodeGen.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/MathExtras.h"
#include <cassert>
#include <cstdint>
#include <utility>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 43-43
```cpp
#include <vector>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 45-45
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 47-50
```cpp
static std::pair<unsigned, unsigned> getMFHiLoOpc(unsigned Src) {
  if (Mips::ACC64RegClass.contains(Src))
    return std::make_pair((unsigned)Mips::PseudoMFHI,
                          (unsigned)Mips::PseudoMFLO);
```
- EN: Implements `getMFHiLoOpc`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMFHiLoOpc`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 52-53
```cpp
  if (Mips::ACC64DSPRegClass.contains(Src))
    return std::make_pair((unsigned)Mips::MFHI_DSP, (unsigned)Mips::MFLO_DSP);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 55-57
```cpp
  if (Mips::ACC128RegClass.contains(Src))
    return std::make_pair((unsigned)Mips::PseudoMFHI64,
                          (unsigned)Mips::PseudoMFLO64);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 59-60
```cpp
  return std::make_pair(0, 0);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 62-62
```cpp
namespace {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 64-68
```cpp
/// Helper class to expand pseudos.
class ExpandPseudo {
public:
  ExpandPseudo(MachineFunction &MF);
  bool expand();
```
- EN: Declares `to`, packaging target-specific state and APIs around `MipsSEFrameLowering`.
- CN: 这里声明 `to`，把与 `MipsSEFrameLowering` 相关的目标特定状态和 API 组织在一起。

### Lines 70-71
```cpp
private:
  using Iter = MachineBasicBlock::iterator;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 73-85
```cpp
  bool expandInstr(MachineBasicBlock &MBB, Iter I);
  void expandLoadCCond(MachineBasicBlock &MBB, Iter I);
  void expandStoreCCond(MachineBasicBlock &MBB, Iter I);
  void expandLoadACC(MachineBasicBlock &MBB, Iter I, unsigned RegSize);
  void expandStoreACC(MachineBasicBlock &MBB, Iter I, unsigned MFHiOpc,
                      unsigned MFLoOpc, unsigned RegSize);
  bool expandCopy(MachineBasicBlock &MBB, Iter I);
  bool expandCopyACC(MachineBasicBlock &MBB, Iter I, unsigned MFHiOpc,
                     unsigned MFLoOpc);
  bool expandBuildPairF64(MachineBasicBlock &MBB,
                          MachineBasicBlock::iterator I, bool FP64) const;
  bool expandExtractElementF64(MachineBasicBlock &MBB,
                               MachineBasicBlock::iterator I, bool FP64) const;
```
- EN: Declares `expandInstr`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandInstr`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 87-92
```cpp
  MachineFunction &MF;
  MachineRegisterInfo &MRI;
  const MipsSubtarget &Subtarget;
  const MipsSEInstrInfo &TII;
  const MipsRegisterInfo &RegInfo;
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 94-94
```cpp
} // end anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 96-100
```cpp
ExpandPseudo::ExpandPseudo(MachineFunction &MF_)
    : MF(MF_), MRI(MF.getRegInfo()),
      Subtarget(MF.getSubtarget<MipsSubtarget>()),
      TII(*static_cast<const MipsSEInstrInfo *>(Subtarget.getInstrInfo())),
      RegInfo(*Subtarget.getRegisterInfo()) {}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 102-103
```cpp
bool ExpandPseudo::expand() {
  bool Expanded = false;
```
- EN: Implements `ExpandPseudo::expand`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ExpandPseudo::expand`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 105-108
```cpp
  for (auto &MBB : MF) {
    for (Iter I = MBB.begin(), End = MBB.end(); I != End;)
      Expanded |= expandInstr(MBB, I++);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 110-111
```cpp
  return Expanded;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 113-127
```cpp
bool ExpandPseudo::expandInstr(MachineBasicBlock &MBB, Iter I) {
  switch(I->getOpcode()) {
  case Mips::LOAD_CCOND_DSP:
    expandLoadCCond(MBB, I);
    break;
  case Mips::STORE_CCOND_DSP:
    expandStoreCCond(MBB, I);
    break;
  case Mips::LOAD_ACC64:
  case Mips::LOAD_ACC64DSP:
    expandLoadACC(MBB, I, 4);
    break;
  case Mips::LOAD_ACC128:
    expandLoadACC(MBB, I, 8);
    break;
```
- EN: Implements `ExpandPseudo::expandInstr`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ExpandPseudo::expandInstr`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 128-142
```cpp
  case Mips::STORE_ACC64:
    expandStoreACC(MBB, I, Mips::PseudoMFHI, Mips::PseudoMFLO, 4);
    break;
  case Mips::STORE_ACC64DSP:
    expandStoreACC(MBB, I, Mips::MFHI_DSP, Mips::MFLO_DSP, 4);
    break;
  case Mips::STORE_ACC128:
    expandStoreACC(MBB, I, Mips::PseudoMFHI64, Mips::PseudoMFLO64, 8);
    break;
  case Mips::BuildPairF64:
    if (expandBuildPairF64(MBB, I, false))
      MBB.erase(I);
    return false;
  case Mips::BuildPairF64_64:
    if (expandBuildPairF64(MBB, I, true))
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 143-157
```cpp
      MBB.erase(I);
    return false;
  case Mips::ExtractElementF64:
    if (expandExtractElementF64(MBB, I, false))
      MBB.erase(I);
    return false;
  case Mips::ExtractElementF64_64:
    if (expandExtractElementF64(MBB, I, true))
      MBB.erase(I);
    return false;
  case TargetOpcode::COPY:
    if (!expandCopy(MBB, I))
      return false;
    break;
  default:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 158-159
```cpp
    return false;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 161-163
```cpp
  MBB.erase(I);
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 165-167
```cpp
void ExpandPseudo::expandLoadCCond(MachineBasicBlock &MBB, Iter I) {
  //  load $vr, FI
  //  copy ccond, $vr
```
- EN: Implements `ExpandPseudo::expandLoadCCond`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ExpandPseudo::expandLoadCCond`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 169-169
```cpp
  assert(I->getOperand(0).isReg() && I->getOperand(1).isFI());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 171-173
```cpp
  const TargetRegisterClass *RC = RegInfo.intRegClass(4);
  Register VR = MRI.createVirtualRegister(RC);
  Register Dst = I->getOperand(0).getReg(), FI = I->getOperand(1).getIndex();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 175-178
```cpp
  TII.loadRegFromStack(MBB, I, VR, FI, RC, 0);
  BuildMI(MBB, I, I->getDebugLoc(), TII.get(TargetOpcode::COPY), Dst)
    .addReg(VR, RegState::Kill);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 180-182
```cpp
void ExpandPseudo::expandStoreCCond(MachineBasicBlock &MBB, Iter I) {
  //  copy $vr, ccond
  //  store $vr, FI
```
- EN: Implements `ExpandPseudo::expandStoreCCond`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ExpandPseudo::expandStoreCCond`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 184-184
```cpp
  assert(I->getOperand(0).isReg() && I->getOperand(1).isFI());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 186-188
```cpp
  const TargetRegisterClass *RC = RegInfo.intRegClass(4);
  Register VR = MRI.createVirtualRegister(RC);
  Register Src = I->getOperand(0).getReg(), FI = I->getOperand(1).getIndex();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 190-193
```cpp
  BuildMI(MBB, I, I->getDebugLoc(), TII.get(TargetOpcode::COPY), VR)
    .addReg(Src, getKillRegState(I->getOperand(0).isKill()));
  TII.storeRegToStack(MBB, I, VR, true, FI, RC, 0);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 195-200
```cpp
void ExpandPseudo::expandLoadACC(MachineBasicBlock &MBB, Iter I,
                                 unsigned RegSize) {
  //  load $vr0, FI
  //  copy lo, $vr0
  //  load $vr1, FI + 4
  //  copy hi, $vr1
```
- EN: Implements `ExpandPseudo::expandLoadACC`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ExpandPseudo::expandLoadACC`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 202-202
```cpp
  assert(I->getOperand(0).isReg() && I->getOperand(1).isFI());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 204-211
```cpp
  const TargetRegisterClass *RC = RegInfo.intRegClass(RegSize);
  Register VR0 = MRI.createVirtualRegister(RC);
  Register VR1 = MRI.createVirtualRegister(RC);
  Register Dst = I->getOperand(0).getReg(), FI = I->getOperand(1).getIndex();
  Register Lo = RegInfo.getSubReg(Dst, Mips::sub_lo);
  Register Hi = RegInfo.getSubReg(Dst, Mips::sub_hi);
  DebugLoc DL = I->getDebugLoc();
  const MCInstrDesc &Desc = TII.get(TargetOpcode::COPY);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 213-217
```cpp
  TII.loadRegFromStack(MBB, I, VR0, FI, RC, 0);
  BuildMI(MBB, I, DL, Desc, Lo).addReg(VR0, RegState::Kill);
  TII.loadRegFromStack(MBB, I, VR1, FI, RC, RegSize);
  BuildMI(MBB, I, DL, Desc, Hi).addReg(VR1, RegState::Kill);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 219-225
```cpp
void ExpandPseudo::expandStoreACC(MachineBasicBlock &MBB, Iter I,
                                  unsigned MFHiOpc, unsigned MFLoOpc,
                                  unsigned RegSize) {
  //  mflo $vr0, src
  //  store $vr0, FI
  //  mfhi $vr1, src
  //  store $vr1, FI + 4
```
- EN: Implements `ExpandPseudo::expandStoreACC`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ExpandPseudo::expandStoreACC`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 227-227
```cpp
  assert(I->getOperand(0).isReg() && I->getOperand(1).isFI());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 229-234
```cpp
  const TargetRegisterClass *RC = RegInfo.intRegClass(RegSize);
  Register VR0 = MRI.createVirtualRegister(RC);
  Register VR1 = MRI.createVirtualRegister(RC);
  Register Src = I->getOperand(0).getReg(), FI = I->getOperand(1).getIndex();
  RegState SrcKill = getKillRegState(I->getOperand(0).isKill());
  DebugLoc DL = I->getDebugLoc();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 236-240
```cpp
  BuildMI(MBB, I, DL, TII.get(MFLoOpc), VR0).addReg(Src);
  TII.storeRegToStack(MBB, I, VR0, true, FI, RC, 0);
  BuildMI(MBB, I, DL, TII.get(MFHiOpc), VR1).addReg(Src, SrcKill);
  TII.storeRegToStack(MBB, I, VR1, true, FI, RC, RegSize);
}
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 242-244
```cpp
bool ExpandPseudo::expandCopy(MachineBasicBlock &MBB, Iter I) {
  Register Src = I->getOperand(1).getReg();
  std::pair<unsigned, unsigned> Opcodes = getMFHiLoOpc(Src);
```
- EN: Implements `ExpandPseudo::expandCopy`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ExpandPseudo::expandCopy`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 246-247
```cpp
  if (!Opcodes.first)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 249-250
```cpp
  return expandCopyACC(MBB, I, Opcodes.first, Opcodes.second);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 252-257
```cpp
bool ExpandPseudo::expandCopyACC(MachineBasicBlock &MBB, Iter I,
                                 unsigned MFHiOpc, unsigned MFLoOpc) {
  //  mflo $vr0, src
  //  copy dst_lo, $vr0
  //  mfhi $vr1, src
  //  copy dst_hi, $vr1
```
- EN: Implements `ExpandPseudo::expandCopyACC`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ExpandPseudo::expandCopyACC`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 259-268
```cpp
  unsigned Dst = I->getOperand(0).getReg(), Src = I->getOperand(1).getReg();
  const TargetRegisterClass *DstRC = RegInfo.getMinimalPhysRegClass(Dst);
  unsigned VRegSize = RegInfo.getRegSizeInBits(*DstRC) / 16;
  const TargetRegisterClass *RC = RegInfo.intRegClass(VRegSize);
  Register VR0 = MRI.createVirtualRegister(RC);
  Register VR1 = MRI.createVirtualRegister(RC);
  RegState SrcKill = getKillRegState(I->getOperand(1).isKill());
  Register DstLo = RegInfo.getSubReg(Dst, Mips::sub_lo);
  Register DstHi = RegInfo.getSubReg(Dst, Mips::sub_hi);
  DebugLoc DL = I->getDebugLoc();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 270-277
```cpp
  BuildMI(MBB, I, DL, TII.get(MFLoOpc), VR0).addReg(Src);
  BuildMI(MBB, I, DL, TII.get(TargetOpcode::COPY), DstLo)
    .addReg(VR0, RegState::Kill);
  BuildMI(MBB, I, DL, TII.get(MFHiOpc), VR1).addReg(Src, SrcKill);
  BuildMI(MBB, I, DL, TII.get(TargetOpcode::COPY), DstHi)
    .addReg(VR1, RegState::Kill);
  return true;
}
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 279-293
```cpp
/// This method expands the same instruction that MipsSEInstrInfo::
/// expandBuildPairF64 does, for the case when ABI is fpxx and mthc1 is not
/// available and the case where the ABI is FP64A. It is implemented here
/// because frame indexes are eliminated before MipsSEInstrInfo::
/// expandBuildPairF64 is called.
bool ExpandPseudo::expandBuildPairF64(MachineBasicBlock &MBB,
                                      MachineBasicBlock::iterator I,
                                      bool FP64) const {
  // For fpxx and when mthc1 is not available, use:
  //   spill + reload via ldc1
  //
  // The case where dmtc1 is available doesn't need to be handled here
  // because it never creates a BuildPairF64 node.
  //
  // The FP64A ABI (fp64 with nooddspreg) must also use a spill/reload sequence
```
- EN: Implements `ExpandPseudo::expandBuildPairF64`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ExpandPseudo::expandBuildPairF64`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 294-307
```cpp
  // for odd-numbered double precision values (because the lower 32-bits is
  // transferred with mtc1 which is redirected to the upper half of the even
  // register). Unfortunately, we have to make this decision before register
  // allocation so for now we use a spill/reload sequence for all
  // double-precision values in regardless of being an odd/even register.
  //
  // For the cases that should be covered here MipsSEISelDAGToDAG adds $sp as
  // implicit operand, so other passes (like ShrinkWrapping) are aware that
  // stack is used.
  if (I->getNumOperands() == 4 && I->getOperand(3).isReg()
      && I->getOperand(3).getReg() == Mips::SP) {
    Register DstReg = I->getOperand(0).getReg();
    Register LoReg = I->getOperand(1).getReg();
    Register HiReg = I->getOperand(2).getReg();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 309-313
```cpp
    // It should be impossible to have FGR64 on MIPS-II or MIPS32r1 (which are
    // the cases where mthc1 is not available). 64-bit architectures and
    // MIPS32r2 or later can use FGR64 though.
    assert(Subtarget.isGP64bit() || Subtarget.hasMTHC1() ||
           !Subtarget.isFP64bit());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 315-317
```cpp
    const TargetRegisterClass *RC = &Mips::GPR32RegClass;
    const TargetRegisterClass *RC2 =
        FP64 ? &Mips::FGR64RegClass : &Mips::AFGR64RegClass;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 319-328
```cpp
    // We re-use the same spill slot each time so that the stack frame doesn't
    // grow too much in functions with a large number of moves.
    int FI = MF.getInfo<MipsFunctionInfo>()->getMoveF64ViaSpillFI(MF, RC2);
    if (!Subtarget.isLittle())
      std::swap(LoReg, HiReg);
    TII.storeRegToStack(MBB, I, LoReg, I->getOperand(1).isKill(), FI, RC, 0);
    TII.storeRegToStack(MBB, I, HiReg, I->getOperand(2).isKill(), FI, RC, 4);
    TII.loadRegFromStack(MBB, I, DstReg, FI, RC2, 0);
    return true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 330-331
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 333-342
```cpp
/// This method expands the same instruction that MipsSEInstrInfo::
/// expandExtractElementF64 does, for the case when ABI is fpxx and mfhc1 is not
/// available and the case where the ABI is FP64A. It is implemented here
/// because frame indexes are eliminated before MipsSEInstrInfo::
/// expandExtractElementF64 is called.
bool ExpandPseudo::expandExtractElementF64(MachineBasicBlock &MBB,
                                           MachineBasicBlock::iterator I,
                                           bool FP64) const {
  const MachineOperand &Op1 = I->getOperand(1);
  const MachineOperand &Op2 = I->getOperand(2);
```
- EN: Implements `ExpandPseudo::expandExtractElementF64`, a mutation/build routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ExpandPseudo::expandExtractElementF64`，它是一个围绕机器基本块展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 344-348
```cpp
  if ((Op1.isReg() && Op1.isUndef()) || (Op2.isReg() && Op2.isUndef())) {
    Register DstReg = I->getOperand(0).getReg();
    BuildMI(MBB, I, I->getDebugLoc(), TII.get(Mips::IMPLICIT_DEF), DstReg);
    return true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 350-364
```cpp
  // For fpxx and when mfhc1 is not available, use:
  //   spill + reload via ldc1
  //
  // The case where dmfc1 is available doesn't need to be handled here
  // because it never creates a ExtractElementF64 node.
  //
  // The FP64A ABI (fp64 with nooddspreg) must also use a spill/reload sequence
  // for odd-numbered double precision values (because the lower 32-bits is
  // transferred with mfc1 which is redirected to the upper half of the even
  // register). Unfortunately, we have to make this decision before register
  // allocation so for now we use a spill/reload sequence for all
  // double-precision values in regardless of being an odd/even register.
  //
  // For the cases that should be covered here MipsSEISelDAGToDAG adds $sp as
  // implicit operand, so other passes (like ShrinkWrapping) are aware that
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 365-371
```cpp
  // stack is used.
  if (I->getNumOperands() == 4 && I->getOperand(3).isReg()
      && I->getOperand(3).getReg() == Mips::SP) {
    Register DstReg = I->getOperand(0).getReg();
    Register SrcReg = Op1.getReg();
    unsigned N = Op2.getImm();
    int64_t Offset = 4 * (Subtarget.isLittle() ? N : (1 - N));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 373-377
```cpp
    // It should be impossible to have FGR64 on MIPS-II or MIPS32r1 (which are
    // the cases where mfhc1 is not available). 64-bit architectures and
    // MIPS32r2 or later can use FGR64 though.
    assert(Subtarget.isGP64bit() || Subtarget.hasMTHC1() ||
           !Subtarget.isFP64bit());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 379-381
```cpp
    const TargetRegisterClass *RC =
        FP64 ? &Mips::FGR64RegClass : &Mips::AFGR64RegClass;
    const TargetRegisterClass *RC2 = &Mips::GPR32RegClass;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 383-389
```cpp
    // We re-use the same spill slot each time so that the stack frame doesn't
    // grow too much in functions with a large number of moves.
    int FI = MF.getInfo<MipsFunctionInfo>()->getMoveF64ViaSpillFI(MF, RC);
    TII.storeRegToStack(MBB, I, SrcReg, Op1.isKill(), FI, RC, 0);
    TII.loadRegFromStack(MBB, I, DstReg, FI, RC2, Offset);
    return true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 391-392
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 394-395
```cpp
MipsSEFrameLowering::MipsSEFrameLowering(const MipsSubtarget &STI)
    : MipsFrameLowering(STI, STI.getStackAlignment()) {}
```
- EN: Implements `MipsSEFrameLowering::MipsSEFrameLowering`, a lowering routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEFrameLowering::MipsSEFrameLowering`，它是一个围绕子目标特性处理展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 397-400
```cpp
void MipsSEFrameLowering::emitPrologue(MachineFunction &MF,
                                       MachineBasicBlock &MBB) const {
  MachineFrameInfo &MFI    = MF.getFrameInfo();
  MipsFunctionInfo *MipsFI = MF.getInfo<MipsFunctionInfo>();
```
- EN: Implements `MipsSEFrameLowering::emitPrologue`, a emission/printing routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEFrameLowering::emitPrologue`，它是一个围绕机器函数状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 402-404
```cpp
  const MipsSEInstrInfo &TII =
      *static_cast<const MipsSEInstrInfo *>(STI.getInstrInfo());
  const MipsRegisterInfo &RegInfo = *STI.getRegisterInfo();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 406-414
```cpp
  MachineBasicBlock::iterator MBBI = MBB.begin();
  DebugLoc dl;
  MipsABIInfo ABI = STI.getABI();
  unsigned SP = ABI.GetStackPtr();
  unsigned FP = ABI.GetFramePtr();
  unsigned ZERO = ABI.GetNullPtr();
  unsigned MOVE = ABI.GetGPRMoveOp();
  unsigned ADDiu = ABI.GetPtrAddiuOp();
  unsigned AND = ABI.IsN64() ? Mips::AND64 : Mips::AND;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 416-417
```cpp
  const TargetRegisterClass *RC = ABI.ArePtrs64bit() ?
        &Mips::GPR64RegClass : &Mips::GPR32RegClass;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 419-420
```cpp
  // First, compute final stack size.
  uint64_t StackSize = MFI.getStackSize();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 422-423
```cpp
  // No need to allocate space on the stack.
  if (StackSize == 0 && !MFI.adjustsStack()) return;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 425-425
```cpp
  CFIInstBuilder CFIBuilder(MBB, MBBI, MachineInstr::NoFlags);
```
- EN: Declares `CFIBuilder`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `CFIBuilder`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 427-429
```cpp
  // Adjust stack.
  TII.adjustStackPtr(SP, -StackSize, MBB, MBBI);
  CFIBuilder.buildDefCFAOffset(StackSize);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 431-432
```cpp
  if (MF.getFunction().hasFnAttribute("interrupt"))
    emitInterruptPrologueStub(MF, MBB);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 434-434
```cpp
  const std::vector<CalleeSavedInfo> &CSI = MFI.getCalleeSavedInfo();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 436-439
```cpp
  // Find the instruction past the last instruction that saves a callee-saved
  // register to the stack.
  std::advance(MBBI, CSI.size());
  CFIBuilder.setInsertPoint(MBBI);
```
- EN: Declares `std::advance`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `std::advance`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 441-446
```cpp
  if (!CSI.empty()) {
    // Iterate over list of callee-saved registers and emit .cfi_offset
    // directives.
    for (const CalleeSavedInfo &I : CSI) {
      int64_t Offset = MFI.getObjectOffset(I.getFrameIdx());
      MCRegister Reg = I.getReg();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 448-452
```cpp
      // If Reg is a double precision register, emit two cfa_offsets,
      // one for each of the paired single precision registers.
      if (Mips::AFGR64RegClass.contains(Reg)) {
        MCRegister Reg0 = RegInfo.getSubReg(Reg, Mips::sub_lo);
        MCRegister Reg1 = RegInfo.getSubReg(Reg, Mips::sub_hi);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 454-455
```cpp
        if (!STI.isLittle())
          std::swap(Reg0, Reg1);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 457-461
```cpp
        CFIBuilder.buildOffset(Reg0, Offset);
        CFIBuilder.buildOffset(Reg1, Offset + 4);
      } else if (Mips::FGR64RegClass.contains(Reg)) {
        MCRegister Reg0 = Reg;
        MCRegister Reg1 = Reg + 1;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 463-464
```cpp
        if (!STI.isLittle())
          std::swap(Reg0, Reg1);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 466-473
```cpp
        CFIBuilder.buildOffset(Reg0, Offset);
        CFIBuilder.buildOffset(Reg1, Offset + 4);
      } else {
        // Reg is either in GPR32 or FGR32.
        CFIBuilder.buildOffset(Reg, Offset);
      }
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 475-482
```cpp
  if (MipsFI->callsEhReturn()) {
    // Insert instructions that spill eh data registers.
    for (int I = 0; I < 4; ++I) {
      if (!MBB.isLiveIn(ABI.GetEhDataReg(I)))
        MBB.addLiveIn(ABI.GetEhDataReg(I));
      TII.storeRegToStackSlot(MBB, MBBI, ABI.GetEhDataReg(I), false,
                              MipsFI->getEhDataRegFI(I), RC, Register());
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 484-489
```cpp
    // Emit .cfi_offset directives for eh data registers.
    for (int I = 0; I < 4; ++I) {
      int64_t Offset = MFI.getObjectOffset(MipsFI->getEhDataRegFI(I));
      CFIBuilder.buildOffset(ABI.GetEhDataReg(I), Offset);
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 491-495
```cpp
  // if framepointer enabled, set it to point to the stack pointer.
  if (hasFP(MF)) {
    // Insert instruction "move $fp, $sp" at this location.
    BuildMI(MBB, MBBI, dl, TII.get(MOVE), FP).addReg(SP).addReg(ZERO)
      .setMIFlag(MachineInstr::FrameSetup);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 497-497
```cpp
    CFIBuilder.buildDefCFARegister(FP);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 499-505
```cpp
    if (RegInfo.hasStackRealignment(MF)) {
      // addiu $Reg, $zero, -MaxAlignment
      // andi $sp, $sp, $Reg
      Register VR = MF.getRegInfo().createVirtualRegister(RC);
      assert((Log2(MFI.getMaxAlign()) < 16) &&
             "Function's alignment size requirement is not supported.");
      int64_t MaxAlign = -(int64_t)MFI.getMaxAlign().value();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 507-508
```cpp
      BuildMI(MBB, MBBI, dl, TII.get(ADDiu), VR).addReg(ZERO).addImm(MaxAlign);
      BuildMI(MBB, MBBI, dl, TII.get(AND), SP).addReg(SP).addReg(VR);
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 510-519
```cpp
      if (hasBP(MF)) {
        // move $s7, $sp
        unsigned BP = STI.isABI_N64() ? Mips::S7_64 : Mips::S7;
        BuildMI(MBB, MBBI, dl, TII.get(MOVE), BP)
          .addReg(SP)
          .addReg(ZERO);
      }
    }
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 521-525
```cpp
void MipsSEFrameLowering::emitInterruptPrologueStub(
    MachineFunction &MF, MachineBasicBlock &MBB) const {
  MipsFunctionInfo *MipsFI = MF.getInfo<MipsFunctionInfo>();
  MachineBasicBlock::iterator MBBI = MBB.begin();
  DebugLoc DL = MBBI != MBB.end() ? MBBI->getDebugLoc() : DebugLoc();
```
- EN: Implements `MipsSEFrameLowering::emitInterruptPrologueStub`, a emission/printing routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEFrameLowering::emitInterruptPrologueStub`，它是一个围绕机器函数状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 527-535
```cpp
  // Report an error the target doesn't support Mips32r2 or later.
  // The epilogue relies on the use of the "ehb" to clear execution
  // hazards. Pre R2 Mips relies on an implementation defined number
  // of "ssnop"s to clear the execution hazard. Support for ssnop hazard
  // clearing is not provided so reject that configuration.
  if (!STI.hasMips32r2())
    report_fatal_error(
        "\"interrupt\" attribute is not supported on pre-MIPS32R2 or "
        "MIPS16 targets.");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 537-543
```cpp
  // The GP register contains the "user" value, so we cannot perform
  // any gp relative loads until we restore the "kernel" or "system" gp
  // value. Until support is written we shall only accept the static
  // relocation model.
  if ((STI.getRelocationModel() != Reloc::Static))
    report_fatal_error("\"interrupt\" attribute is only supported for the "
                       "static relocation model on MIPS at the present time.");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 545-547
```cpp
  if (!STI.isABI_O32() || STI.hasMips64())
    report_fatal_error("\"interrupt\" attribute is only supported for the "
                       "O32 ABI on MIPS32R2+ at the present time.");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 549-552
```cpp
  // Perform ISR handling like GCC
  StringRef IntKind =
      MF.getFunction().getFnAttribute("interrupt").getValueAsString();
  const TargetRegisterClass *PtrRC = &Mips::GPR32RegClass;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 554-562
```cpp
  // EIC interrupt handling needs to read the Cause register to disable
  // interrupts.
  if (IntKind == "eic") {
    // Coprocessor registers are always live per se.
    MBB.addLiveIn(Mips::COP013);
    BuildMI(MBB, MBBI, DL, STI.getInstrInfo()->get(Mips::MFC0), Mips::K0)
        .addReg(Mips::COP013)
        .addImm(0)
        .setMIFlag(MachineInstr::FrameSetup);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 564-569
```cpp
    BuildMI(MBB, MBBI, DL, STI.getInstrInfo()->get(Mips::EXT), Mips::K0)
        .addReg(Mips::K0)
        .addImm(10)
        .addImm(6)
        .setMIFlag(MachineInstr::FrameSetup);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 571-576
```cpp
  // Fetch and spill EPC
  MBB.addLiveIn(Mips::COP014);
  BuildMI(MBB, MBBI, DL, STI.getInstrInfo()->get(Mips::MFC0), Mips::K1)
      .addReg(Mips::COP014)
      .addImm(0)
      .setMIFlag(MachineInstr::FrameSetup);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 578-579
```cpp
  STI.getInstrInfo()->storeRegToStack(MBB, MBBI, Mips::K1, false,
                                      MipsFI->getISRRegFI(0), PtrRC, 0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 581-586
```cpp
  // Fetch and Spill Status
  MBB.addLiveIn(Mips::COP012);
  BuildMI(MBB, MBBI, DL, STI.getInstrInfo()->get(Mips::MFC0), Mips::K1)
      .addReg(Mips::COP012)
      .addImm(0)
      .setMIFlag(MachineInstr::FrameSetup);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 588-589
```cpp
  STI.getInstrInfo()->storeRegToStack(MBB, MBBI, Mips::K1, false,
                                      MipsFI->getISRRegFI(1), PtrRC, 0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 591-595
```cpp
  // Build the configuration for disabling lower priority interrupts. Non EIC
  // interrupts need to be masked off with zero, EIC from the Cause register.
  unsigned InsPosition = 8;
  unsigned InsSize = 0;
  unsigned SrcReg = Mips::ZERO;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 597-611
```cpp
  // If the interrupt we're tied to is the EIC, switch the source for the
  // masking off interrupts to the cause register.
  if (IntKind == "eic") {
    SrcReg = Mips::K0;
    InsPosition = 10;
    InsSize = 6;
  } else
    InsSize = StringSwitch<unsigned>(IntKind)
                  .Case("sw0", 1)
                  .Case("sw1", 2)
                  .Case("hw0", 3)
                  .Case("hw1", 4)
                  .Case("hw2", 5)
                  .Case("hw3", 6)
                  .Case("hw4", 7)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 612-614
```cpp
                  .Case("hw5", 8)
                  .Default(0);
  assert(InsSize != 0 && "Unknown interrupt type!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 616-621
```cpp
  BuildMI(MBB, MBBI, DL, STI.getInstrInfo()->get(Mips::INS), Mips::K1)
      .addReg(SrcReg)
      .addImm(InsPosition)
      .addImm(InsSize)
      .addReg(Mips::K1)
      .setMIFlag(MachineInstr::FrameSetup);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 623-629
```cpp
  // Mask off KSU, ERL, EXL
  BuildMI(MBB, MBBI, DL, STI.getInstrInfo()->get(Mips::INS), Mips::K1)
      .addReg(Mips::ZERO)
      .addImm(1)
      .addImm(4)
      .addReg(Mips::K1)
      .setMIFlag(MachineInstr::FrameSetup);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 631-638
```cpp
  // Disable the FPU as we are not spilling those register sets.
  if (!STI.useSoftFloat())
    BuildMI(MBB, MBBI, DL, STI.getInstrInfo()->get(Mips::INS), Mips::K1)
        .addReg(Mips::ZERO)
        .addImm(29)
        .addImm(1)
        .addReg(Mips::K1)
        .setMIFlag(MachineInstr::FrameSetup);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 640-645
```cpp
  // Set the new status
  BuildMI(MBB, MBBI, DL, STI.getInstrInfo()->get(Mips::MTC0), Mips::COP012)
      .addReg(Mips::K1)
      .addImm(0)
      .setMIFlag(MachineInstr::FrameSetup);
}
```
- EN: Declares `BuildMI`, a mutation/build routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕机器指令展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 647-651
```cpp
void MipsSEFrameLowering::emitEpilogue(MachineFunction &MF,
                                       MachineBasicBlock &MBB) const {
  MachineBasicBlock::iterator MBBI = MBB.getFirstTerminator();
  MachineFrameInfo &MFI            = MF.getFrameInfo();
  MipsFunctionInfo *MipsFI = MF.getInfo<MipsFunctionInfo>();
```
- EN: Implements `MipsSEFrameLowering::emitEpilogue`, a emission/printing routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEFrameLowering::emitEpilogue`，它是一个围绕机器函数状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 653-654
```cpp
  const MipsSEInstrInfo &TII =
      *static_cast<const MipsSEInstrInfo *>(STI.getInstrInfo());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 656-661
```cpp
  DebugLoc DL = MBBI != MBB.end() ? MBBI->getDebugLoc() : DebugLoc();
  MipsABIInfo ABI = STI.getABI();
  unsigned SP = ABI.GetStackPtr();
  unsigned FP = ABI.GetFramePtr();
  unsigned ZERO = ABI.GetNullPtr();
  unsigned MOVE = ABI.GetGPRMoveOp();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 663-666
```cpp
  // if framepointer enabled, restore the stack pointer.
  if (hasFP(MF)) {
    // Find the first instruction that restores a callee-saved register.
    MachineBasicBlock::iterator I = MBBI;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 668-669
```cpp
    for (unsigned i = 0; i < MFI.getCalleeSavedInfo().size(); ++i)
      --I;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 671-673
```cpp
    // Insert instruction "move $sp, $fp" at this location.
    BuildMI(MBB, I, DL, TII.get(MOVE), SP).addReg(FP).addReg(ZERO);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 675-677
```cpp
  if (MipsFI->callsEhReturn()) {
    const TargetRegisterClass *RC =
        ABI.ArePtrs64bit() ? &Mips::GPR64RegClass : &Mips::GPR32RegClass;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 679-682
```cpp
    // Find first instruction that restores a callee-saved register.
    MachineBasicBlock::iterator I = MBBI;
    for (unsigned i = 0; i < MFI.getCalleeSavedInfo().size(); ++i)
      --I;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 684-689
```cpp
    // Insert instructions that restore eh data registers.
    for (int J = 0; J < 4; ++J) {
      TII.loadRegFromStackSlot(MBB, I, ABI.GetEhDataReg(J),
                               MipsFI->getEhDataRegFI(J), RC, Register());
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 691-692
```cpp
  if (MF.getFunction().hasFnAttribute("interrupt"))
    emitInterruptEpilogueStub(MF, MBB);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 694-695
```cpp
  // Get the number of bytes from FrameInfo
  uint64_t StackSize = MFI.getStackSize();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 697-698
```cpp
  if (!StackSize)
    return;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 700-702
```cpp
  // Adjust stack.
  TII.adjustStackPtr(SP, StackSize, MBB, MBBI);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 704-708
```cpp
void MipsSEFrameLowering::emitInterruptEpilogueStub(
    MachineFunction &MF, MachineBasicBlock &MBB) const {
  MachineBasicBlock::iterator MBBI = MBB.getLastNonDebugInstr();
  MipsFunctionInfo *MipsFI = MF.getInfo<MipsFunctionInfo>();
  DebugLoc DL = MBBI != MBB.end() ? MBBI->getDebugLoc() : DebugLoc();
```
- EN: Implements `MipsSEFrameLowering::emitInterruptEpilogueStub`, a emission/printing routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEFrameLowering::emitInterruptEpilogueStub`，它是一个围绕机器函数状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 710-711
```cpp
  // Perform ISR handling like GCC
  const TargetRegisterClass *PtrRC = &Mips::GPR32RegClass;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 713-715
```cpp
  // Disable Interrupts.
  BuildMI(MBB, MBBI, DL, STI.getInstrInfo()->get(Mips::DI), Mips::ZERO);
  BuildMI(MBB, MBBI, DL, STI.getInstrInfo()->get(Mips::EHB));
```
- EN: Declares `BuildMI`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BuildMI`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 717-722
```cpp
  // Restore EPC
  STI.getInstrInfo()->loadRegFromStackSlot(
      MBB, MBBI, Mips::K1, MipsFI->getISRRegFI(0), PtrRC, Register());
  BuildMI(MBB, MBBI, DL, STI.getInstrInfo()->get(Mips::MTC0), Mips::COP014)
      .addReg(Mips::K1)
      .addImm(0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 724-730
```cpp
  // Restore Status
  STI.getInstrInfo()->loadRegFromStackSlot(
      MBB, MBBI, Mips::K1, MipsFI->getISRRegFI(1), PtrRC, Register());
  BuildMI(MBB, MBBI, DL, STI.getInstrInfo()->get(Mips::MTC0), Mips::COP012)
      .addReg(Mips::K1)
      .addImm(0);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 732-736
```cpp
StackOffset
MipsSEFrameLowering::getFrameIndexReference(const MachineFunction &MF, int FI,
                                            Register &FrameReg) const {
  const MachineFrameInfo &MFI = MF.getFrameInfo();
  MipsABIInfo ABI = STI.getABI();
```
- EN: Implements `MipsSEFrameLowering::getFrameIndexReference`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEFrameLowering::getFrameIndexReference`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 738-741
```cpp
  if (MFI.isFixedObjectIndex(FI))
    FrameReg = hasFP(MF) ? ABI.GetFramePtr() : ABI.GetStackPtr();
  else
    FrameReg = hasBP(MF) ? ABI.GetBasePtr() : ABI.GetStackPtr();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 743-746
```cpp
  return StackOffset::getFixed(MFI.getObjectOffset(FI) + MFI.getStackSize() -
                               getOffsetOfLocalArea() +
                               MFI.getOffsetAdjustment());
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 748-752
```cpp
bool MipsSEFrameLowering::spillCalleeSavedRegisters(
    MachineBasicBlock &MBB, MachineBasicBlock::iterator MI,
    ArrayRef<CalleeSavedInfo> CSI, const TargetRegisterInfo *TRI) const {
  MachineFunction *MF = MBB.getParent();
  const TargetInstrInfo &TII = *STI.getInstrInfo();
```
- EN: Implements `MipsSEFrameLowering::spillCalleeSavedRegisters`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEFrameLowering::spillCalleeSavedRegisters`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 754-764
```cpp
  for (const CalleeSavedInfo &I : CSI) {
    // Add the callee-saved register as live-in. Do not add if the register is
    // RA and return address is taken, because it has already been added in
    // method MipsTargetLowering::lowerRETURNADDR.
    // It's killed at the spill, unless the register is RA and return address
    // is taken.
    MCRegister Reg = I.getReg();
    bool IsRAAndRetAddrIsTaken = (Reg == Mips::RA || Reg == Mips::RA_64)
        && MF->getFrameInfo().isReturnAddressTaken();
    if (!IsRAAndRetAddrIsTaken)
      MBB.addLiveIn(Reg);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 766-772
```cpp
    // ISRs require HI/LO to be spilled into kernel registers to be then
    // spilled to the stack frame.
    bool IsLOHI = (Reg == Mips::LO0 || Reg == Mips::LO0_64 ||
                   Reg == Mips::HI0 || Reg == Mips::HI0_64);
    const Function &Func = MBB.getParent()->getFunction();
    if (IsLOHI && Func.hasFnAttribute("interrupt")) {
      DebugLoc DL = MI->getDebugLoc();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 774-784
```cpp
      unsigned Op = 0;
      if (!STI.getABI().ArePtrs64bit()) {
        Op = (Reg == Mips::HI0) ? Mips::MFHI : Mips::MFLO;
        Reg = Mips::K0;
      } else {
        Op = (Reg == Mips::HI0) ? Mips::MFHI64 : Mips::MFLO64;
        Reg = Mips::K0_64;
      }
      BuildMI(MBB, MI, DL, TII.get(Op), Mips::K0)
          .setMIFlag(MachineInstr::FrameSetup);
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 786-791
```cpp
    // Insert the spill to the stack frame.
    bool IsKill = !IsRAAndRetAddrIsTaken;
    const TargetRegisterClass *RC = TRI->getMinimalPhysRegClass(Reg);
    TII.storeRegToStackSlot(MBB, MI, Reg, IsKill, I.getFrameIdx(), RC,
                            Register());
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 793-794
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 796-805
```cpp
bool
MipsSEFrameLowering::hasReservedCallFrame(const MachineFunction &MF) const {
  const MachineFrameInfo &MFI = MF.getFrameInfo();
  // Reserve call frame if the size of the maximum call frame fits into 16-bit
  // immediate field and there are no variable sized objects on the stack.
  // Make sure the second register scavenger spill slot can be accessed with one
  // instruction.
  return isInt<16>(MFI.getMaxCallFrameSize() + getStackAlignment()) &&
    !MFI.hasVarSizedObjects();
}
```
- EN: Implements `MipsSEFrameLowering::hasReservedCallFrame`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEFrameLowering::hasReservedCallFrame`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 807-813
```cpp
/// Mark \p Reg and all registers aliasing it in the bitset.
static void setAliasRegs(MachineFunction &MF, BitVector &SavedRegs,
                         unsigned Reg) {
  const TargetRegisterInfo *TRI = MF.getSubtarget().getRegisterInfo();
  for (MCRegAliasIterator AI(Reg, TRI, true); AI.isValid(); ++AI)
    SavedRegs.set(*AI);
}
```
- EN: Implements `setAliasRegs`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `setAliasRegs`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 815-824
```cpp
void MipsSEFrameLowering::determineCalleeSaves(MachineFunction &MF,
                                               BitVector &SavedRegs,
                                               RegScavenger *RS) const {
  TargetFrameLowering::determineCalleeSaves(MF, SavedRegs, RS);
  const TargetRegisterInfo *TRI = MF.getSubtarget().getRegisterInfo();
  MipsFunctionInfo *MipsFI = MF.getInfo<MipsFunctionInfo>();
  MipsABIInfo ABI = STI.getABI();
  unsigned RA = ABI.IsN64() ? Mips::RA_64 : Mips::RA;
  unsigned FP = ABI.GetFramePtr();
  unsigned BP = ABI.IsN64() ? Mips::S7_64 : Mips::S7;
```
- EN: Implements `MipsSEFrameLowering::determineCalleeSaves`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsSEFrameLowering::determineCalleeSaves`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 826-833
```cpp
  // Mark $ra and $fp as used if function has dedicated frame pointer.
  if (hasFP(MF)) {
    setAliasRegs(MF, SavedRegs, RA);
    setAliasRegs(MF, SavedRegs, FP);
  }
  // Mark $s7 as used if function has dedicated base pointer.
  if (hasBP(MF))
    setAliasRegs(MF, SavedRegs, BP);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 835-837
```cpp
  // Create spill slots for eh data registers if function calls eh_return.
  if (MipsFI->callsEhReturn())
    MipsFI->createEhDataRegsFI(MF);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 839-841
```cpp
  // Create spill slots for Coprocessor 0 registers if function is an ISR.
  if (MipsFI->isISR())
    MipsFI->createISRRegFI(MF);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 843-854
```cpp
  // Expand pseudo instructions which load, store or copy accumulators.
  // Add an emergency spill slot if a pseudo was expanded.
  if (ExpandPseudo(MF).expand()) {
    // The spill slot should be half the size of the accumulator. If target have
    // general-purpose registers 64 bits wide, it should be 64-bit, otherwise
    // it should be 32-bit.
    const TargetRegisterClass &RC = STI.isGP64bit() ?
      Mips::GPR64RegClass : Mips::GPR32RegClass;
    int FI = MF.getFrameInfo().CreateSpillStackObject(TRI->getSpillSize(RC),
                                                      TRI->getSpillAlign(RC));
    RS->addScavengingFrameIndex(FI);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 856-857
```cpp
  // Set scavenging frame index if necessary.
  uint64_t MaxSPOffset = estimateStackSize(MF);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 859-863
```cpp
  // MSA has a minimum offset of 10 bits signed. If there is a variable
  // sized object on the stack, the estimation cannot account for it.
  if (isIntN(STI.hasMSA() ? 10 : 16, MaxSPOffset) &&
      !MF.getFrameInfo().hasVarSizedObjects())
    return;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 865-870
```cpp
  const TargetRegisterClass &RC =
      ABI.ArePtrs64bit() ? Mips::GPR64RegClass : Mips::GPR32RegClass;
  int FI = MF.getFrameInfo().CreateSpillStackObject(TRI->getSpillSize(RC),
                                                    TRI->getSpillAlign(RC));
  RS->addScavengingFrameIndex(FI);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 872-875
```cpp
const MipsFrameLowering *
llvm::createMipsSEFrameLowering(const MipsSubtarget &ST) {
  return new MipsSEFrameLowering(ST);
}
```
- EN: Implements `llvm::createMipsSEFrameLowering`, a lowering routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createMipsSEFrameLowering`，它是一个围绕子目标特性处理展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: stack-frame layout and prologue/epilogue lowering.
  - CN: 核心职责：栈帧布局以及序言/结语生成。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MipsSEFrameLowering.h`, `MCTargetDesc/MipsABIInfo.h`, `MipsMachineFunction.h`, `MipsRegisterInfo.h`, `MipsSEInstrInfo.h`, `MipsSubtarget.h`.
  - CN: 后端本地头文件：`MipsSEFrameLowering.h`, `MCTargetDesc/MipsABIInfo.h`, `MipsMachineFunction.h`, `MipsRegisterInfo.h`, `MipsSEInstrInfo.h`, `MipsSubtarget.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/BitVector.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/StringSwitch.h`, `llvm/CodeGen/CFIInstBuilder.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstr.h` ... (+13 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/BitVector.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/StringSwitch.h`, `llvm/CodeGen/CFIInstBuilder.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstr.h` ... (+13 more)。
- EN: Standard/system headers: `cassert`, `cstdint`, `utility`, `vector`.
  - CN: 标准库/系统头文件：`cassert`, `cstdint`, `utility`, `vector`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
