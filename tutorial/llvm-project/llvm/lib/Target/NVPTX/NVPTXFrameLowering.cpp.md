# NVPTXFrameLowering.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXFrameLowering.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains the NVPTX implementation of TargetFrameLowering class.
- 目的（中文）: 该文件实现 LLVM 目标后端中的相关逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //=======- NVPTXFrameLowering.cpp - NVPTX Frame Information ---*- C++ -*-=====//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file contains the NVPTX implementation of TargetFrameLowering class.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #include "NVPTXFrameLowering.h"
14: #include "NVPTX.h"
15: #include "NVPTXRegisterInfo.h"
16: #include "NVPTXSubtarget.h"
17: #include "NVPTXTargetMachine.h"
18: #include "llvm/CodeGen/MachineFrameInfo.h"
19: #include "llvm/CodeGen/MachineFunction.h"
20: #include "llvm/CodeGen/MachineInstrBuilder.h"
21: #include "llvm/CodeGen/MachineRegisterInfo.h"
22: #include "llvm/CodeGen/TargetInstrInfo.h"
23:
24: using namespace llvm;
25:
26: NVPTXFrameLowering::NVPTXFrameLowering()
27:     : TargetFrameLowering(TargetFrameLowering::StackGrowsUp, Align(8), 0) {}
28:
29: bool NVPTXFrameLowering::hasFPImpl(const MachineFunction &MF) const {
30:   return true;
31: }
32:
33: void NVPTXFrameLowering::emitPrologue(MachineFunction &MF,
34:                                       MachineBasicBlock &MBB) const {
35:   if (MF.getFrameInfo().hasStackObjects()) {
36:     assert(&MF.front() == &MBB && "Shrink-wrapping not yet supported");
37:     MachineBasicBlock::iterator MBBI = MBB.begin();
38:     MachineRegisterInfo &MR = MF.getRegInfo();
39:
40:     const NVPTXRegisterInfo *NRI =
```
- EN: This range implements operational logic in helpers such as NVPTXFrameLowering::NVPTXFrameLowering, TargetFrameLowering, NVPTXFrameLowering::hasFPImpl, assert, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXFrameLowering::NVPTXFrameLowering、TargetFrameLowering、NVPTXFrameLowering::hasFPImpl、assert 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 41-80
```cpp
41:         MF.getSubtarget<NVPTXSubtarget>().getRegisterInfo();
42:
43:     // This instruction really occurs before first instruction
44:     // in the BB, so giving it no debug location.
45:     DebugLoc dl = DebugLoc();
46:
47:     // Emits
48:     //   mov %SPL, %depot;
49:     //   cvta.local %SP, %SPL;
50:     // for local address accesses in MF.
51:     bool Is64Bit =
52:         static_cast<const NVPTXTargetMachine &>(MF.getTarget()).is64Bit();
53:     unsigned CvtaLocalOpcode =
54:         (Is64Bit ? NVPTX::cvta_local_64 : NVPTX::cvta_local);
55:     unsigned MovDepotOpcode =
56:         (Is64Bit ? NVPTX::MOV_DEPOT_ADDR_64 : NVPTX::MOV_DEPOT_ADDR);
57:     if (!MR.use_empty(NRI->getFrameRegister(MF))) {
58:       // If %SP is not used, do not bother emitting "cvta.local %SP, %SPL".
59:       MBBI = BuildMI(MBB, MBBI, dl,
60:                      MF.getSubtarget().getInstrInfo()->get(CvtaLocalOpcode),
61:                      NRI->getFrameRegister(MF))
62:                  .addReg(NRI->getFrameLocalRegister(MF));
63:     }
64:     if (!MR.use_empty(NRI->getFrameLocalRegister(MF))) {
65:       BuildMI(MBB, MBBI, dl,
66:               MF.getSubtarget().getInstrInfo()->get(MovDepotOpcode),
67:               NRI->getFrameLocalRegister(MF))
68:           .addImm(MF.getFunctionNumber());
69:     }
70:   }
71: }
72:
73: StackOffset
74: NVPTXFrameLowering::getFrameIndexReference(const MachineFunction &MF, int FI,
75:                                            Register &FrameReg) const {
76:   const MachineFrameInfo &MFI = MF.getFrameInfo();
77:   FrameReg = NVPTX::VRDepot;
78:   return StackOffset::getFixed(MFI.getObjectOffset(FI) -
79:                                getOffsetOfLocalArea());
80: }
```
- EN: This range implements operational logic in helpers such as getRegisterInfo, DebugLoc, getTarget, getFrameRegister, translating backend policy into executable code.
- CN: 这一段实现了 getRegisterInfo、DebugLoc、getTarget、getFrameRegister 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-101
```cpp
 81:
 82: void NVPTXFrameLowering::emitEpilogue(MachineFunction &MF,
 83:                                       MachineBasicBlock &MBB) const {}
 84:
 85: // This function eliminates ADJCALLSTACKDOWN,
 86: // ADJCALLSTACKUP pseudo instructions
 87: MachineBasicBlock::iterator NVPTXFrameLowering::eliminateCallFramePseudoInstr(
 88:     MachineFunction &MF, MachineBasicBlock &MBB,
 89:     MachineBasicBlock::iterator I) const {
 90:   // Simply discard ADJCALLSTACKDOWN,
 91:   // ADJCALLSTACKUP instructions.
 92:   return MBB.erase(I);
 93: }
 94:
 95: TargetFrameLowering::DwarfFrameBase
 96: NVPTXFrameLowering::getDwarfFrameBase(const MachineFunction &MF) const {
 97:   DwarfFrameBase FrameBase;
 98:   FrameBase.Kind = DwarfFrameBase::CFA;
 99:   FrameBase.Location.Offset = 0;
100:   return FrameBase;
101: }
```
- EN: This range implements operational logic in helpers such as erase, NVPTXFrameLowering::getDwarfFrameBase, translating backend policy into executable code.
- CN: 这一段实现了 erase、NVPTXFrameLowering::getDwarfFrameBase 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Lowering turns higher-level IR constructs into forms accepted by the target pipeline.
  - CN: 降级过程会把更高层的 IR 构造转换成目标流水线可接受的形式。
- EN: Frame lowering controls stack frame creation, prologue/epilogue structure, and calling-convention details.
  - CN: 栈帧降级负责栈帧建立、序言/尾声结构以及调用约定细节。
- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include NVPTXFrameLowering::NVPTXFrameLowering, TargetFrameLowering, NVPTXFrameLowering::hasFPImpl, assert, begin, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 NVPTXFrameLowering::NVPTXFrameLowering, TargetFrameLowering, NVPTXFrameLowering::hasFPImpl, assert, begin，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTXFrameLowering.h`
  - `NVPTX.h`
  - `NVPTXRegisterInfo.h`
  - `NVPTXSubtarget.h`
  - `NVPTXTargetMachine.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/CodeGen/MachineFrameInfo.h`
  - `llvm/CodeGen/MachineFunction.h`
  - `llvm/CodeGen/MachineInstrBuilder.h`
  - `llvm/CodeGen/MachineRegisterInfo.h`
  - `llvm/CodeGen/TargetInstrInfo.h`
