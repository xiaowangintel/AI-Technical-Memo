# M68kExpandPseudo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/M68kExpandPseudo.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Provides target-specific implementation details for the M68k backend.
- **用途 (CN)**: 提供 M68k 后端的目标相关实现细节。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- M68kExpandPseudo.cpp - Expand pseudo instructions -------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// This file contains a pass that expands pseudo instructions into target
  11: /// instructions to allow proper scheduling, if-conversion, other late
  12: /// optimizations, or simply the encoding of the instructions.
  13: ///
  14: //===----------------------------------------------------------------------===//
  15: 
  16: #include "M68k.h"
  17: #include "M68kFrameLowering.h"
  18: #include "M68kInstrInfo.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `M68k.h`, `M68kFrameLowering.h`, `M68kInstrInfo.h` that expose the LLVM and target interfaces used in later logic. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `M68k.h`, `M68kFrameLowering.h`, `M68kInstrInfo.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "M68kMachineFunction.h"
  20: #include "M68kSubtarget.h"
  21: 
  22: #include "llvm/CodeGen/MachineFunctionPass.h"
  23: #include "llvm/CodeGen/MachineInstrBuilder.h"
  24: #include "llvm/CodeGen/MachineRegisterInfo.h"
  25: #include "llvm/CodeGen/Passes.h" // For IDs of passes that are preserved.
  26: #include "llvm/IR/EHPersonalities.h"
  27: #include "llvm/IR/GlobalValue.h"
  28: 
  29: using namespace llvm;
  30: 
  31: #define DEBUG_TYPE "m68k-expand-pseudo"
  32: #define PASS_NAME "M68k pseudo instruction expansion pass"
  33: 
  34: namespace {
  35: class M68kExpandPseudo : public MachineFunctionPass {
  36: public:
```
- **EN**: It imports dependencies such as `M68kMachineFunction.h`, `M68kSubtarget.h`, `MachineFunctionPass.h`, `MachineInstrBuilder.h`, `MachineRegisterInfo.h`, `Passes.h` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `M68kExpandPseudo`. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `M68kMachineFunction.h`, `M68kSubtarget.h`, `MachineFunctionPass.h`, `MachineInstrBuilder.h`, `MachineRegisterInfo.h`, `Passes.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `M68kExpandPseudo` 等 TableGen 记录。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 37-54 / 第 37-54 行
```cpp
  37:   static char ID;
  38:   M68kExpandPseudo() : MachineFunctionPass(ID) {}
  39: 
  40:   void getAnalysisUsage(AnalysisUsage &AU) const override {
  41:     AU.setPreservesCFG();
  42:     AU.addPreservedID(MachineLoopInfoID);
  43:     AU.addPreservedID(MachineDominatorsID);
  44:     MachineFunctionPass::getAnalysisUsage(AU);
  45:   }
  46: 
  47:   const M68kSubtarget *STI;
  48:   const M68kInstrInfo *TII;
  49:   const M68kRegisterInfo *TRI;
  50:   const M68kMachineFunctionInfo *MFI;
  51:   const M68kFrameLowering *FL;
  52: 
  53:   bool runOnMachineFunction(MachineFunction &Fn) override;
  54: 
```
- **EN**: The range implements or declares functions including `M68kExpandPseudo`.
- **CN**: 这一段实现或声明了 `M68kExpandPseudo` 等函数。

### Lines 55-72 / 第 55-72 行
```cpp
  55:   MachineFunctionProperties getRequiredProperties() const override {
  56:     return MachineFunctionProperties().setNoVRegs();
  57:   }
  58: 
  59: private:
  60:   bool ExpandMI(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI);
  61:   bool ExpandMBB(MachineBasicBlock &MBB);
  62: };
  63: char M68kExpandPseudo::ID = 0;
  64: } // End anonymous namespace.
  65: 
  66: INITIALIZE_PASS(M68kExpandPseudo, DEBUG_TYPE, PASS_NAME, false, false)
  67: 
  68: /// If \p MBBI is a pseudo instruction, this method expands
  69: /// it to the corresponding (sequence of) actual instruction(s).
  70: /// \returns true if \p MBBI has been expanded.
  71: bool M68kExpandPseudo::ExpandMI(MachineBasicBlock &MBB,
  72:                                 MachineBasicBlock::iterator MBBI) {
```
- **EN**: The range implements or declares functions including `M68kExpandPseudo::ExpandMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kExpandPseudo::ExpandMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 73-90 / 第 73-90 行
```cpp
  73:   MachineInstr &MI = *MBBI;
  74:   MachineInstrBuilder MIB(*MI.getParent()->getParent(), MI);
  75:   unsigned Opcode = MI.getOpcode();
  76:   DebugLoc DL = MBBI->getDebugLoc();
  77:   /// TODO infer argument size to create less switch cases
  78:   switch (Opcode) {
  79:   default:
  80:     return false;
  81: 
  82:   case M68k::MOVI8di:
  83:     return TII->ExpandMOVI(MIB, MVT::i8);
  84:   case M68k::MOVI16ri:
  85:     return TII->ExpandMOVI(MIB, MVT::i16);
  86:   case M68k::MOVI32ri:
  87:     return TII->ExpandMOVI(MIB, MVT::i32);
  88: 
  89:   case M68k::MOVXd16d8:
  90:     return TII->ExpandMOVX_RR(MIB, MVT::i16, MVT::i8);
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 91-108 / 第 91-108 行
```cpp
  91:   case M68k::MOVXd32d8:
  92:     return TII->ExpandMOVX_RR(MIB, MVT::i32, MVT::i8);
  93:   case M68k::MOVXd32d16:
  94:     return TII->ExpandMOVX_RR(MIB, MVT::i32, MVT::i16);
  95: 
  96:   case M68k::MOVSXd16d8:
  97:     return TII->ExpandMOVSZX_RR(MIB, true, MVT::i16, MVT::i8);
  98:   case M68k::MOVSXd32d8:
  99:     return TII->ExpandMOVSZX_RR(MIB, true, MVT::i32, MVT::i8);
 100:   case M68k::MOVSXd32d16:
 101:     return TII->ExpandMOVSZX_RR(MIB, true, MVT::i32, MVT::i16);
 102: 
 103:   case M68k::MOVZXd16d8:
 104:     return TII->ExpandMOVSZX_RR(MIB, false, MVT::i16, MVT::i8);
 105:   case M68k::MOVZXd32d8:
 106:     return TII->ExpandMOVSZX_RR(MIB, false, MVT::i32, MVT::i8);
 107:   case M68k::MOVZXd32d16:
 108:     return TII->ExpandMOVSZX_RR(MIB, false, MVT::i32, MVT::i16);
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 109-126 / 第 109-126 行
```cpp
 109: 
 110:   case M68k::MOVSXd16j8:
 111:     return TII->ExpandMOVSZX_RM(MIB, true, TII->get(M68k::MOV8dj), MVT::i16,
 112:                                 MVT::i8);
 113:   case M68k::MOVSXd32j8:
 114:     return TII->ExpandMOVSZX_RM(MIB, true, TII->get(M68k::MOV8dj), MVT::i32,
 115:                                 MVT::i8);
 116:   case M68k::MOVSXd32j16:
 117:     return TII->ExpandMOVSZX_RM(MIB, true, TII->get(M68k::MOV16rj), MVT::i32,
 118:                                 MVT::i16);
 119: 
 120:   case M68k::MOVZXd16j8:
 121:     return TII->ExpandMOVSZX_RM(MIB, false, TII->get(M68k::MOV8dj), MVT::i16,
 122:                                 MVT::i8);
 123:   case M68k::MOVZXd32j8:
 124:     return TII->ExpandMOVSZX_RM(MIB, false, TII->get(M68k::MOV8dj), MVT::i32,
 125:                                 MVT::i8);
 126:   case M68k::MOVZXd32j16:
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 127-144 / 第 127-144 行
```cpp
 127:     return TII->ExpandMOVSZX_RM(MIB, false, TII->get(M68k::MOV16rj), MVT::i32,
 128:                                 MVT::i16);
 129: 
 130:   case M68k::MOVSXd16p8:
 131:     return TII->ExpandMOVSZX_RM(MIB, true, TII->get(M68k::MOV8dp), MVT::i16,
 132:                                 MVT::i8);
 133:   case M68k::MOVSXd32p8:
 134:     return TII->ExpandMOVSZX_RM(MIB, true, TII->get(M68k::MOV8dp), MVT::i32,
 135:                                 MVT::i8);
 136:   case M68k::MOVSXd32p16:
 137:     return TII->ExpandMOVSZX_RM(MIB, true, TII->get(M68k::MOV16rp), MVT::i32,
 138:                                 MVT::i16);
 139: 
 140:   case M68k::MOVZXd16p8:
 141:     return TII->ExpandMOVSZX_RM(MIB, false, TII->get(M68k::MOV8dp), MVT::i16,
 142:                                 MVT::i8);
 143:   case M68k::MOVZXd32p8:
 144:     return TII->ExpandMOVSZX_RM(MIB, false, TII->get(M68k::MOV8dp), MVT::i32,
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 145-162 / 第 145-162 行
```cpp
 145:                                 MVT::i8);
 146:   case M68k::MOVZXd32p16:
 147:     return TII->ExpandMOVSZX_RM(MIB, false, TII->get(M68k::MOV16rp), MVT::i32,
 148:                                 MVT::i16);
 149: 
 150:   case M68k::MOVSXd16f8:
 151:     return TII->ExpandMOVSZX_RM(MIB, true, TII->get(M68k::MOV8df), MVT::i16,
 152:                                 MVT::i8);
 153:   case M68k::MOVSXd32f8:
 154:     return TII->ExpandMOVSZX_RM(MIB, true, TII->get(M68k::MOV8df), MVT::i32,
 155:                                 MVT::i8);
 156:   case M68k::MOVSXd32f16:
 157:     return TII->ExpandMOVSZX_RM(MIB, true, TII->get(M68k::MOV16rf), MVT::i32,
 158:                                 MVT::i16);
 159: 
 160:   case M68k::MOVZXd16f8:
 161:     return TII->ExpandMOVSZX_RM(MIB, false, TII->get(M68k::MOV8df), MVT::i16,
 162:                                 MVT::i8);
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 163-180 / 第 163-180 行
```cpp
 163:   case M68k::MOVZXd32f8:
 164:     return TII->ExpandMOVSZX_RM(MIB, false, TII->get(M68k::MOV8df), MVT::i32,
 165:                                 MVT::i8);
 166:   case M68k::MOVZXd32f16:
 167:     return TII->ExpandMOVSZX_RM(MIB, false, TII->get(M68k::MOV16rf), MVT::i32,
 168:                                 MVT::i16);
 169: 
 170:   case M68k::MOVSXd16q8:
 171:     return TII->ExpandMOVSZX_RM(MIB, true, TII->get(M68k::MOV8dq), MVT::i16,
 172:                                 MVT::i8);
 173:   case M68k::MOVSXd32q8:
 174:     return TII->ExpandMOVSZX_RM(MIB, true, TII->get(M68k::MOV8dq), MVT::i32,
 175:                                 MVT::i8);
 176:   case M68k::MOVSXd32q16:
 177:     return TII->ExpandMOVSZX_RM(MIB, true, TII->get(M68k::MOV16dq), MVT::i32,
 178:                                 MVT::i16);
 179: 
 180:   case M68k::MOVZXd16q8:
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 181-198 / 第 181-198 行
```cpp
 181:     return TII->ExpandMOVSZX_RM(MIB, false, TII->get(M68k::MOV8dq), MVT::i16,
 182:                                 MVT::i8);
 183:   case M68k::MOVZXd32q8:
 184:     return TII->ExpandMOVSZX_RM(MIB, false, TII->get(M68k::MOV8dq), MVT::i32,
 185:                                 MVT::i8);
 186:   case M68k::MOVZXd32q16:
 187:     return TII->ExpandMOVSZX_RM(MIB, false, TII->get(M68k::MOV16dq), MVT::i32,
 188:                                 MVT::i16);
 189: 
 190:   case M68k::MOVM16jm_P:
 191:     return TII->ExpandMOVEM(MIB, TII->get(M68k::MOVM16jm), /*IsRM=*/false);
 192:   case M68k::MOVM32jm_P:
 193:     return TII->ExpandMOVEM(MIB, TII->get(M68k::MOVM32jm), /*IsRM=*/false);
 194: 
 195:   case M68k::MOVM16pm_P:
 196:     return TII->ExpandMOVEM(MIB, TII->get(M68k::MOVM16pm), /*IsRM=*/false);
 197:   case M68k::MOVM32pm_P:
 198:     return TII->ExpandMOVEM(MIB, TII->get(M68k::MOVM32pm), /*IsRM=*/false);
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 199-216 / 第 199-216 行
```cpp
 199: 
 200:   case M68k::MOVM16mj_P:
 201:     return TII->ExpandMOVEM(MIB, TII->get(M68k::MOVM16mj), /*IsRM=*/true);
 202:   case M68k::MOVM32mj_P:
 203:     return TII->ExpandMOVEM(MIB, TII->get(M68k::MOVM32mj), /*IsRM=*/true);
 204: 
 205:   case M68k::MOVM16mp_P:
 206:     return TII->ExpandMOVEM(MIB, TII->get(M68k::MOVM16mp), /*IsRM=*/true);
 207:   case M68k::MOVM32mp_P:
 208:     return TII->ExpandMOVEM(MIB, TII->get(M68k::MOVM32mp), /*IsRM=*/true);
 209: 
 210:   case M68k::TCRETURNq:
 211:   case M68k::TCRETURNj: {
 212:     MachineOperand &JumpTarget = MI.getOperand(0);
 213:     MachineOperand &StackAdjust = MI.getOperand(1);
 214:     assert(StackAdjust.isImm() && "Expecting immediate value.");
 215: 
 216:     // Adjust stack pointer.
```
- **EN**: The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 217-234 / 第 217-234 行
```cpp
 217:     int StackAdj = StackAdjust.getImm();
 218:     int MaxTCDelta = MFI->getTCReturnAddrDelta();
 219:     int Offset = 0;
 220:     assert(MaxTCDelta <= 0 && "MaxTCDelta should never be positive");
 221: 
 222:     // Incoporate the retaddr area.
 223:     Offset = StackAdj - MaxTCDelta;
 224:     assert(Offset >= 0 && "Offset should never be negative");
 225: 
 226:     if (Offset) {
 227:       // Check for possible merge with preceding ADD instruction.
 228:       Offset += FL->mergeSPUpdates(MBB, MBBI, true);
 229:       FL->emitSPUpdate(MBB, MBBI, Offset, /*InEpilogue=*/true);
 230:     }
 231: 
 232:     // Jump to label or value in register.
 233:     if (Opcode == M68k::TCRETURNq) {
 234:       MachineInstrBuilder MIB =
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 235-252 / 第 235-252 行
```cpp
 235:           BuildMI(MBB, MBBI, DL, TII->get(M68k::TAILJMPq));
 236:       if (JumpTarget.isGlobal()) {
 237:         MIB.addGlobalAddress(JumpTarget.getGlobal(), JumpTarget.getOffset(),
 238:                              JumpTarget.getTargetFlags());
 239:       } else {
 240:         assert(JumpTarget.isSymbol());
 241:         MIB.addExternalSymbol(JumpTarget.getSymbolName(),
 242:                               JumpTarget.getTargetFlags());
 243:       }
 244:     } else {
 245:       BuildMI(MBB, MBBI, DL, TII->get(M68k::TAILJMPj))
 246:           .addReg(JumpTarget.getReg(), RegState::Kill);
 247:     }
 248: 
 249:     MachineInstr &NewMI = *std::prev(MBBI);
 250:     NewMI.copyImplicitOps(*MBBI->getParent()->getParent(), *MBBI);
 251: 
 252:     // Delete the pseudo instruction TCRETURN.
```
- **EN**: The range implements or declares functions including `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 253-270 / 第 253-270 行
```cpp
 253:     MBB.erase(MBBI);
 254: 
 255:     return true;
 256:   }
 257:   case M68k::RET: {
 258:     if (MBB.getParent()->getFunction().getCallingConv() ==
 259:         CallingConv::M68k_INTR) {
 260:       BuildMI(MBB, MBBI, DL, TII->get(M68k::RTE));
 261:     } else if (int64_t StackAdj = MBBI->getOperand(0).getImm(); StackAdj == 0) {
 262:       BuildMI(MBB, MBBI, DL, TII->get(M68k::RTS));
 263:     } else {
 264:       // Copy return address from stack to a free address(A0 or A1) register
 265:       // TODO check if pseudo expand uses free address register
 266:       BuildMI(MBB, MBBI, DL, TII->get(M68k::MOV32aj), M68k::A1)
 267:           .addReg(M68k::SP);
 268: 
 269:       // Adjust SP
 270:       FL->emitSPUpdate(MBB, MBBI, StackAdj, /*InEpilogue=*/true);
```
- **EN**: The range implements or declares functions including `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 271-288 / 第 271-288 行
```cpp
 271: 
 272:       // Put the return address on stack
 273:       BuildMI(MBB, MBBI, DL, TII->get(M68k::MOV32ja))
 274:           .addReg(M68k::SP)
 275:           .addReg(M68k::A1);
 276: 
 277:       // RTS
 278:       BuildMI(MBB, MBBI, DL, TII->get(M68k::RTS));
 279:     }
 280: 
 281:     // FIXME: Can rest of the operands be ignored, if there is any?
 282:     MBB.erase(MBBI);
 283:     return true;
 284:   }
 285:   }
 286:   llvm_unreachable("Previous switch has a fallthrough?");
 287: }
 288: 
```
- **EN**: The range implements or declares functions including `BuildMI`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 289-306 / 第 289-306 行
```cpp
 289: /// Expand all pseudo instructions contained in \p MBB.
 290: /// \returns true if any expansion occurred for \p MBB.
 291: bool M68kExpandPseudo::ExpandMBB(MachineBasicBlock &MBB) {
 292:   bool Modified = false;
 293: 
 294:   // MBBI may be invalidated by the expansion.
 295:   MachineBasicBlock::iterator MBBI = MBB.begin(), E = MBB.end();
 296:   while (MBBI != E) {
 297:     MachineBasicBlock::iterator NMBBI = std::next(MBBI);
 298:     Modified |= ExpandMI(MBB, MBBI);
 299:     MBBI = NMBBI;
 300:   }
 301: 
 302:   return Modified;
 303: }
 304: 
 305: bool M68kExpandPseudo::runOnMachineFunction(MachineFunction &MF) {
 306:   STI = &MF.getSubtarget<M68kSubtarget>();
```
- **EN**: The range implements or declares functions including `M68kExpandPseudo::ExpandMBB`, `M68kExpandPseudo::runOnMachineFunction`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `M68kExpandPseudo::ExpandMBB`, `M68kExpandPseudo::runOnMachineFunction` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 307-321 / 第 307-321 行
```cpp
 307:   TII = STI->getInstrInfo();
 308:   TRI = STI->getRegisterInfo();
 309:   MFI = MF.getInfo<M68kMachineFunctionInfo>();
 310:   FL = STI->getFrameLowering();
 311: 
 312:   bool Modified = false;
 313:   for (MachineBasicBlock &MBB : MF)
 314:     Modified |= ExpandMBB(MBB);
 315:   return Modified;
 316: }
 317: 
 318: /// Returns an instance of the pseudo instruction expansion pass.
 319: FunctionPass *llvm::createM68kExpandPseudoPass() {
 320:   return new M68kExpandPseudo();
 321: }
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Calling convention**: Defines how arguments, returns, and preserved registers are assigned. / 定义参数、返回值和保留寄存器的分配方式。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `M68k.h`
- `M68kFrameLowering.h`
- `M68kInstrInfo.h`
- `M68kMachineFunction.h`
- `M68kSubtarget.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/CodeGen/Passes.h`
- `llvm/IR/EHPersonalities.h`
- `llvm/IR/GlobalValue.h`
