# M68kInstrInfo.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/M68kInstrInfo.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file defines or implements target instruction information for the M68k backend.
- **用途 (CN)**: 该文件用于 M68k 后端，负责定义或实现目标指令信息。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- M68kInstrInfo.h - M68k Instruction Information ----------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// This file contains the M68k implementation of the TargetInstrInfo class.
  11: ///
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #ifndef LLVM_LIB_TARGET_M68K_M68KINSTRINFO_H
  15: #define LLVM_LIB_TARGET_M68K_M68KINSTRINFO_H
  16: 
  17: #include "M68k.h"
  18: #include "M68kRegisterInfo.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `M68k.h`, `M68kRegisterInfo.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `M68k.h`, `M68kRegisterInfo.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 19-36 / 第 19-36 行
```cpp
  19: 
  20: #include "MCTargetDesc/M68kBaseInfo.h"
  21: 
  22: #include "llvm/CodeGen/MachineInstrBuilder.h"
  23: #include "llvm/CodeGen/TargetInstrInfo.h"
  24: 
  25: #define GET_INSTRINFO_HEADER
  26: #include "M68kGenInstrInfo.inc"
  27: 
  28: namespace llvm {
  29: 
  30: class M68kSubtarget;
  31: 
  32: namespace M68k {
  33: // These MUST be kept in sync with codes definitions in M68kInstrInfo.td
  34: enum CondCode {
  35:   COND_T = 0,   // True
  36:   COND_F = 1,   // False
```
- **EN**: It imports dependencies such as `M68kBaseInfo.h`, `MachineInstrBuilder.h`, `TargetInstrInfo.h`, `M68kGenInstrInfo.inc` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `M68kSubtarget`. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `M68kBaseInfo.h`, `MachineInstrBuilder.h`, `TargetInstrInfo.h`, `M68kGenInstrInfo.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `M68kSubtarget` 等 TableGen 记录。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 37-54 / 第 37-54 行
```cpp
  37:   COND_HI = 2,  // High
  38:   COND_LS = 3,  // Less or Same
  39:   COND_CC = 4,  // Carry Clear
  40:   COND_CS = 5,  // Carry Set
  41:   COND_NE = 6,  // Not Equal
  42:   COND_EQ = 7,  // Equal
  43:   COND_VC = 8,  // Overflow Clear
  44:   COND_VS = 9,  // Overflow Set
  45:   COND_PL = 10, // Plus
  46:   COND_MI = 11, // Minus
  47:   COND_GE = 12, // Greater or Equal
  48:   COND_LT = 13, // Less Than
  49:   COND_GT = 14, // Greater Than
  50:   COND_LE = 15, // Less or Equal
  51:   LAST_VALID_COND = COND_LE,
  52:   COND_INVALID
  53: };
  54: 
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 55-72 / 第 55-72 行
```cpp
  55: // FIXME would be nice tablegen to generate these predicates and converters
  56: // mb tag based
  57: 
  58: static inline M68k::CondCode GetOppositeBranchCondition(M68k::CondCode CC) {
  59:   switch (CC) {
  60:   default:
  61:     llvm_unreachable("Illegal condition code!");
  62:   case M68k::COND_T:
  63:     return M68k::COND_F;
  64:   case M68k::COND_F:
  65:     return M68k::COND_T;
  66:   case M68k::COND_HI:
  67:     return M68k::COND_LS;
  68:   case M68k::COND_LS:
  69:     return M68k::COND_HI;
  70:   case M68k::COND_CC:
  71:     return M68k::COND_CS;
  72:   case M68k::COND_CS:
```
- **EN**: The range implements or declares functions including `GetOppositeBranchCondition`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `GetOppositeBranchCondition` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 73-90 / 第 73-90 行
```cpp
  73:     return M68k::COND_CC;
  74:   case M68k::COND_NE:
  75:     return M68k::COND_EQ;
  76:   case M68k::COND_EQ:
  77:     return M68k::COND_NE;
  78:   case M68k::COND_VC:
  79:     return M68k::COND_VS;
  80:   case M68k::COND_VS:
  81:     return M68k::COND_VC;
  82:   case M68k::COND_PL:
  83:     return M68k::COND_MI;
  84:   case M68k::COND_MI:
  85:     return M68k::COND_PL;
  86:   case M68k::COND_GE:
  87:     return M68k::COND_LT;
  88:   case M68k::COND_LT:
  89:     return M68k::COND_GE;
  90:   case M68k::COND_GT:
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 91-108 / 第 91-108 行
```cpp
  91:     return M68k::COND_LE;
  92:   case M68k::COND_LE:
  93:     return M68k::COND_GT;
  94:   }
  95: }
  96: 
  97: static inline unsigned GetCondBranchFromCond(M68k::CondCode CC) {
  98:   switch (CC) {
  99:   default:
 100:     llvm_unreachable("Illegal condition code!");
 101:   case M68k::COND_EQ:
 102:     return M68k::Beq8;
 103:   case M68k::COND_NE:
 104:     return M68k::Bne8;
 105:   case M68k::COND_LT:
 106:     return M68k::Blt8;
 107:   case M68k::COND_LE:
 108:     return M68k::Ble8;
```
- **EN**: The range implements or declares functions including `GetCondBranchFromCond`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `GetCondBranchFromCond` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 109-126 / 第 109-126 行
```cpp
 109:   case M68k::COND_GT:
 110:     return M68k::Bgt8;
 111:   case M68k::COND_GE:
 112:     return M68k::Bge8;
 113:   case M68k::COND_CS:
 114:     return M68k::Bcs8;
 115:   case M68k::COND_LS:
 116:     return M68k::Bls8;
 117:   case M68k::COND_HI:
 118:     return M68k::Bhi8;
 119:   case M68k::COND_CC:
 120:     return M68k::Bcc8;
 121:   case M68k::COND_MI:
 122:     return M68k::Bmi8;
 123:   case M68k::COND_PL:
 124:     return M68k::Bpl8;
 125:   case M68k::COND_VS:
 126:     return M68k::Bvs8;
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 127-144 / 第 127-144 行
```cpp
 127:   case M68k::COND_VC:
 128:     return M68k::Bvc8;
 129:   }
 130: }
 131: 
 132: static inline M68k::CondCode GetCondFromBranchOpc(unsigned Opcode) {
 133:   switch (Opcode) {
 134:   default:
 135:     return M68k::COND_INVALID;
 136:   case M68k::Beq8:
 137:     return M68k::COND_EQ;
 138:   case M68k::Bne8:
 139:     return M68k::COND_NE;
 140:   case M68k::Blt8:
 141:     return M68k::COND_LT;
 142:   case M68k::Ble8:
 143:     return M68k::COND_LE;
 144:   case M68k::Bgt8:
```
- **EN**: The range implements or declares functions including `GetCondFromBranchOpc`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `GetCondFromBranchOpc` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 145-162 / 第 145-162 行
```cpp
 145:     return M68k::COND_GT;
 146:   case M68k::Bge8:
 147:     return M68k::COND_GE;
 148:   case M68k::Bcs8:
 149:     return M68k::COND_CS;
 150:   case M68k::Bls8:
 151:     return M68k::COND_LS;
 152:   case M68k::Bhi8:
 153:     return M68k::COND_HI;
 154:   case M68k::Bcc8:
 155:     return M68k::COND_CC;
 156:   case M68k::Bmi8:
 157:     return M68k::COND_MI;
 158:   case M68k::Bpl8:
 159:     return M68k::COND_PL;
 160:   case M68k::Bvs8:
 161:     return M68k::COND_VS;
 162:   case M68k::Bvc8:
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 163-180 / 第 163-180 行
```cpp
 163:     return M68k::COND_VC;
 164:   }
 165: }
 166: 
 167: static inline unsigned IsCMP(unsigned Op) {
 168:   switch (Op) {
 169:   default:
 170:     return false;
 171:   case M68k::CMP8dd:
 172:   case M68k::CMP8df:
 173:   case M68k::CMP8di:
 174:   case M68k::CMP8dj:
 175:   case M68k::CMP8dp:
 176:   case M68k::CMP16dr:
 177:   case M68k::CMP16df:
 178:   case M68k::CMP16di:
 179:   case M68k::CMP16dj:
 180:   case M68k::CMP16dp:
```
- **EN**: The range implements or declares functions including `IsCMP`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `IsCMP` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 181-198 / 第 181-198 行
```cpp
 181:     return true;
 182:   }
 183: }
 184: 
 185: static inline bool IsSETCC(unsigned SETCC) {
 186:   switch (SETCC) {
 187:   default:
 188:     return false;
 189:   case M68k::SETd8eq:
 190:   case M68k::SETd8ne:
 191:   case M68k::SETd8lt:
 192:   case M68k::SETd8ge:
 193:   case M68k::SETd8le:
 194:   case M68k::SETd8gt:
 195:   case M68k::SETd8cs:
 196:   case M68k::SETd8cc:
 197:   case M68k::SETd8ls:
 198:   case M68k::SETd8hi:
```
- **EN**: The range implements or declares functions including `IsSETCC`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `IsSETCC` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 199-216 / 第 199-216 行
```cpp
 199:   case M68k::SETd8pl:
 200:   case M68k::SETd8mi:
 201:   case M68k::SETd8vc:
 202:   case M68k::SETd8vs:
 203:   case M68k::SETj8eq:
 204:   case M68k::SETj8ne:
 205:   case M68k::SETj8lt:
 206:   case M68k::SETj8ge:
 207:   case M68k::SETj8le:
 208:   case M68k::SETj8gt:
 209:   case M68k::SETj8cs:
 210:   case M68k::SETj8cc:
 211:   case M68k::SETj8ls:
 212:   case M68k::SETj8hi:
 213:   case M68k::SETj8pl:
 214:   case M68k::SETj8mi:
 215:   case M68k::SETj8vc:
 216:   case M68k::SETj8vs:
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 217-234 / 第 217-234 行
```cpp
 217:   case M68k::SETp8eq:
 218:   case M68k::SETp8ne:
 219:   case M68k::SETp8lt:
 220:   case M68k::SETp8ge:
 221:   case M68k::SETp8le:
 222:   case M68k::SETp8gt:
 223:   case M68k::SETp8cs:
 224:   case M68k::SETp8cc:
 225:   case M68k::SETp8ls:
 226:   case M68k::SETp8hi:
 227:   case M68k::SETp8pl:
 228:   case M68k::SETp8mi:
 229:   case M68k::SETp8vc:
 230:   case M68k::SETp8vs:
 231:     return true;
 232:   }
 233: }
 234: 
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 235-252 / 第 235-252 行
```cpp
 235: } // namespace M68k
 236: 
 237: class M68kInstrInfo : public M68kGenInstrInfo {
 238:   virtual void anchor();
 239: 
 240: protected:
 241:   const M68kSubtarget &Subtarget;
 242:   const M68kRegisterInfo RI;
 243: 
 244: public:
 245:   explicit M68kInstrInfo(const M68kSubtarget &STI);
 246: 
 247:   static const M68kInstrInfo *create(M68kSubtarget &STI);
 248: 
 249:   /// TargetInstrInfo is a superset of MRegister info. As such, whenever a
 250:   /// client has an instance of instruction info, it should always be able to
 251:   /// get register info as well (through this method).
 252:   const M68kRegisterInfo &getRegisterInfo() const { return RI; };
```
- **EN**: This block declares or refines TableGen records such as `M68kInstrInfo`.
- **CN**: 该代码块声明或细化了 `M68kInstrInfo` 等 TableGen 记录。

### Lines 253-270 / 第 253-270 行
```cpp
 253: 
 254:   bool analyzeBranch(MachineBasicBlock &MBB, MachineBasicBlock *&TBB,
 255:                      MachineBasicBlock *&FBB,
 256:                      SmallVectorImpl<MachineOperand> &Cond,
 257:                      bool AllowModify) const override;
 258: 
 259:   bool AnalyzeBranchImpl(MachineBasicBlock &MBB, MachineBasicBlock *&TBB,
 260:                          MachineBasicBlock *&FBB,
 261:                          SmallVectorImpl<MachineOperand> &Cond,
 262:                          bool AllowModify) const;
 263: 
 264:   unsigned removeBranch(MachineBasicBlock &MBB,
 265:                         int *BytesRemoved = nullptr) const override;
 266: 
 267:   unsigned insertBranch(MachineBasicBlock &MBB, MachineBasicBlock *TBB,
 268:                         MachineBasicBlock *FBB, ArrayRef<MachineOperand> Cond,
 269:                         const DebugLoc &DL,
 270:                         int *BytesAdded = nullptr) const override;
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 271-288 / 第 271-288 行
```cpp
 271: 
 272:   void copyPhysReg(MachineBasicBlock &MBB, MachineBasicBlock::iterator MI,
 273:                    const DebugLoc &DL, Register DestReg, Register SrcReg,
 274:                    bool KillSrc, bool RenamableDest = false,
 275:                    bool RenamableSrc = false) const override;
 276: 
 277:   bool getStackSlotRange(const TargetRegisterClass *RC, unsigned SubIdx,
 278:                          unsigned &Size, unsigned &Offset,
 279:                          const MachineFunction &MF) const override;
 280: 
 281:   void storeRegToStackSlot(
 282:       MachineBasicBlock &MBB, MachineBasicBlock::iterator MI, Register SrcReg,
 283:       bool IsKill, int FrameIndex, const TargetRegisterClass *RC, Register VReg,
 284:       MachineInstr::MIFlag Flags = MachineInstr::NoFlags) const override;
 285: 
 286:   void loadRegFromStackSlot(
 287:       MachineBasicBlock &MBB, MachineBasicBlock::iterator MI, Register DestReg,
 288:       int FrameIndex, const TargetRegisterClass *RC, Register VReg,
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 289-306 / 第 289-306 行
```cpp
 289:       unsigned SubReg = 0,
 290:       MachineInstr::MIFlag Flags = MachineInstr::NoFlags) const override;
 291: 
 292:   bool expandPostRAPseudo(MachineInstr &MI) const override;
 293: 
 294:   bool isPCRelRegisterOperandLegal(const MachineOperand &MO) const override;
 295: 
 296:   /// Add appropriate SExt nodes
 297:   void AddSExt(MachineBasicBlock &MBB, MachineBasicBlock::iterator I,
 298:                DebugLoc DL, unsigned Reg, MVT From, MVT To) const;
 299: 
 300:   /// Add appropriate ZExt nodes
 301:   void AddZExt(MachineBasicBlock &MBB, MachineBasicBlock::iterator I,
 302:                DebugLoc DL, unsigned Reg, MVT From, MVT To) const;
 303: 
 304:   /// Move immediate to register
 305:   bool ExpandMOVI(MachineInstrBuilder &MIB, MVT MVTSize) const;
 306: 
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 307-324 / 第 307-324 行
```cpp
 307:   /// Move across register classes without extension
 308:   bool ExpandMOVX_RR(MachineInstrBuilder &MIB, MVT MVTDst, MVT MVTSrc) const;
 309: 
 310:   /// Move from register and extend
 311:   bool ExpandMOVSZX_RR(MachineInstrBuilder &MIB, bool IsSigned, MVT MVTDst,
 312:                        MVT MVTSrc) const;
 313: 
 314:   /// Move from memory and extend
 315:   bool ExpandMOVSZX_RM(MachineInstrBuilder &MIB, bool IsSigned,
 316:                        const MCInstrDesc &Desc, MVT MVTDst, MVT MVTSrc) const;
 317: 
 318:   /// Push/Pop to/from stack
 319:   bool ExpandPUSH_POP(MachineInstrBuilder &MIB, const MCInstrDesc &Desc,
 320:                       bool IsPush) const;
 321: 
 322:   /// Expand all MOVEM pseudos into real MOVEMs
 323:   bool ExpandMOVEM(MachineInstrBuilder &MIB, const MCInstrDesc &Desc,
 324:                    bool IsRM) const;
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 325-340 / 第 325-340 行
```cpp
 325: 
 326:   /// Return a virtual register initialized with the global base register
 327:   /// value. Output instructions required to initialize the register in the
 328:   /// function entry block, if necessary.
 329:   unsigned getGlobalBaseReg(MachineFunction *MF) const;
 330: 
 331:   std::pair<unsigned, unsigned>
 332:   decomposeMachineOperandsTargetFlags(unsigned TF) const override;
 333: 
 334:   ArrayRef<std::pair<unsigned, const char *>>
 335:   getSerializableDirectMachineOperandTargetFlags() const override;
 336: };
 337: 
 338: } // namespace llvm
 339: 
 340: #endif // LLVM_LIB_TARGET_M68K_M68KINSTRINFO_H
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `M68k.h`
- `M68kRegisterInfo.h`
- `MCTargetDesc/M68kBaseInfo.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/TargetInstrInfo.h`
- `M68kGenInstrInfo.inc`
