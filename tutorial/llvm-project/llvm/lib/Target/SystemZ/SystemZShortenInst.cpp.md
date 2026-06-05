# SystemZShortenInst.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZShortenInst.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Provides target-specific implementation details for the SystemZ backend.
- **用途 (CN)**: 提供 SystemZ 后端的目标相关实现细节。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- SystemZShortenInst.cpp - Instruction-shortening pass --------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This pass tries to replace instructions with shorter forms.  For example,
  10: // IILF can be replaced with LLILL or LLILH if the constant fits and if the
  11: // other 32 bits of the GR64 destination are not live.
  12: //
  13: //===----------------------------------------------------------------------===//
  14: 
  15: #include "SystemZTargetMachine.h"
  16: #include "llvm/CodeGen/LiveRegUnits.h"
  17: #include "llvm/CodeGen/MachineFunctionPass.h"
  18: #include "llvm/CodeGen/MachineInstrBuilder.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZTargetMachine.h`, `LiveRegUnits.h`, `MachineFunctionPass.h`, `MachineInstrBuilder.h` that expose the LLVM and target interfaces used in later logic. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZTargetMachine.h`, `LiveRegUnits.h`, `MachineFunctionPass.h`, `MachineInstrBuilder.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "llvm/CodeGen/TargetRegisterInfo.h"
  20: 
  21: using namespace llvm;
  22: 
  23: #define DEBUG_TYPE "systemz-shorten-inst"
  24: 
  25: namespace {
  26: class SystemZShortenInst : public MachineFunctionPass {
  27: public:
  28:   static char ID;
  29:   SystemZShortenInst();
  30: 
  31:   bool processBlock(MachineBasicBlock &MBB);
  32:   bool runOnMachineFunction(MachineFunction &F) override;
  33:   MachineFunctionProperties getRequiredProperties() const override {
  34:     return MachineFunctionProperties().setNoVRegs();
  35:   }
  36: 
```
- **EN**: It imports dependencies such as `TargetRegisterInfo.h` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `SystemZShortenInst`. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `TargetRegisterInfo.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `SystemZShortenInst` 等 TableGen 记录。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 37-54 / 第 37-54 行
```cpp
  37: private:
  38:   bool shortenIIF(MachineInstr &MI, unsigned LLIxL, unsigned LLIxH);
  39:   bool shortenOn0(MachineInstr &MI, unsigned Opcode);
  40:   bool shortenOn01(MachineInstr &MI, unsigned Opcode);
  41:   bool shortenOn001(MachineInstr &MI, unsigned Opcode);
  42:   bool shortenOn001AddCC(MachineInstr &MI, unsigned Opcode);
  43:   bool shortenFPConv(MachineInstr &MI, unsigned Opcode);
  44:   bool shortenFusedFPOp(MachineInstr &MI, unsigned Opcode);
  45: 
  46:   const SystemZInstrInfo *TII;
  47:   const TargetRegisterInfo *TRI;
  48:   LiveRegUnits LiveRegs;
  49: };
  50: 
  51: char SystemZShortenInst::ID = 0;
  52: } // end anonymous namespace
  53: 
  54: INITIALIZE_PASS(SystemZShortenInst, DEBUG_TYPE,
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 55-72 / 第 55-72 行
```cpp
  55:                 "SystemZ Instruction Shortening", false, false)
  56: 
  57: FunctionPass *llvm::createSystemZShortenInstPass(SystemZTargetMachine &TM) {
  58:   return new SystemZShortenInst();
  59: }
  60: 
  61: SystemZShortenInst::SystemZShortenInst()
  62:     : MachineFunctionPass(ID), TII(nullptr) {}
  63: 
  64: // Tie operands if MI has become a two-address instruction.
  65: static void tieOpsIfNeeded(MachineInstr &MI) {
  66:   if (MI.getDesc().getOperandConstraint(1, MCOI::TIED_TO) == 0 &&
  67:       !MI.getOperand(0).isTied())
  68:     MI.tieOperands(0, 1);
  69: }
  70: 
  71: // MI loads one word of a GPR using an IIxF instruction and LLIxL and LLIxH
  72: // are the halfword immediate loads for the same word.  Try to use one of them
```
- **EN**: The range implements or declares functions including `MachineFunctionPass`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `MachineFunctionPass` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 73-90 / 第 73-90 行
```cpp
  73: // instead of IIxF.
  74: bool SystemZShortenInst::shortenIIF(MachineInstr &MI, unsigned LLIxL,
  75:                                     unsigned LLIxH) {
  76:   Register Reg = MI.getOperand(0).getReg();
  77:   // The new opcode will clear the other half of the GR64 reg, so
  78:   // cancel if that is live.
  79:   unsigned thisSubRegIdx =
  80:       (SystemZ::GRH32BitRegClass.contains(Reg) ? SystemZ::subreg_h32
  81:                                                : SystemZ::subreg_l32);
  82:   unsigned otherSubRegIdx =
  83:       (thisSubRegIdx == SystemZ::subreg_l32 ? SystemZ::subreg_h32
  84:                                             : SystemZ::subreg_l32);
  85:   unsigned GR64BitReg =
  86:       TRI->getMatchingSuperReg(Reg, thisSubRegIdx, &SystemZ::GR64BitRegClass);
  87:   Register OtherReg = TRI->getSubReg(GR64BitReg, otherSubRegIdx);
  88:   if (!LiveRegs.available(OtherReg))
  89:     return false;
  90: 
```
- **EN**: The range implements or declares functions including `SystemZShortenInst::shortenIIF`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZShortenInst::shortenIIF` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 91-108 / 第 91-108 行
```cpp
  91:   uint64_t Imm = MI.getOperand(1).getImm();
  92:   if (SystemZ::isImmLL(Imm)) {
  93:     MI.setDesc(TII->get(LLIxL));
  94:     MI.getOperand(0).setReg(SystemZMC::getRegAsGR64(Reg));
  95:     return true;
  96:   }
  97:   if (SystemZ::isImmLH(Imm)) {
  98:     MI.setDesc(TII->get(LLIxH));
  99:     MI.getOperand(0).setReg(SystemZMC::getRegAsGR64(Reg));
 100:     MI.getOperand(1).setImm(Imm >> 16);
 101:     return true;
 102:   }
 103:   return false;
 104: }
 105: 
 106: // Change MI's opcode to Opcode if register operand 0 has a 4-bit encoding.
 107: bool SystemZShortenInst::shortenOn0(MachineInstr &MI, unsigned Opcode) {
 108:   if (SystemZMC::getFirstReg(MI.getOperand(0).getReg()) < 16) {
```
- **EN**: The range implements or declares functions including `SystemZShortenInst::shortenOn0`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZShortenInst::shortenOn0` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 109-126 / 第 109-126 行
```cpp
 109:     MI.setDesc(TII->get(Opcode));
 110:     return true;
 111:   }
 112:   return false;
 113: }
 114: 
 115: // Change MI's opcode to Opcode if register operands 0 and 1 have a
 116: // 4-bit encoding.
 117: bool SystemZShortenInst::shortenOn01(MachineInstr &MI, unsigned Opcode) {
 118:   if (SystemZMC::getFirstReg(MI.getOperand(0).getReg()) < 16 &&
 119:       SystemZMC::getFirstReg(MI.getOperand(1).getReg()) < 16) {
 120:     MI.setDesc(TII->get(Opcode));
 121:     return true;
 122:   }
 123:   return false;
 124: }
 125: 
 126: // Change MI's opcode to Opcode if register operands 0, 1 and 2 have a
```
- **EN**: The range implements or declares functions including `SystemZShortenInst::shortenOn01`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZShortenInst::shortenOn01` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 127-144 / 第 127-144 行
```cpp
 127: // 4-bit encoding and if operands 0 and 1 are tied. Also ties op 0
 128: // with op 1, if MI becomes 2-address.
 129: bool SystemZShortenInst::shortenOn001(MachineInstr &MI, unsigned Opcode) {
 130:   if (SystemZMC::getFirstReg(MI.getOperand(0).getReg()) < 16 &&
 131:       MI.getOperand(1).getReg() == MI.getOperand(0).getReg() &&
 132:       SystemZMC::getFirstReg(MI.getOperand(2).getReg()) < 16) {
 133:     MI.setDesc(TII->get(Opcode));
 134:     tieOpsIfNeeded(MI);
 135:     return true;
 136:   }
 137:   return false;
 138: }
 139: 
 140: // Calls shortenOn001 if CCLive is false. CC def operand is added in
 141: // case of success.
 142: bool SystemZShortenInst::shortenOn001AddCC(MachineInstr &MI, unsigned Opcode) {
 143:   if (LiveRegs.available(SystemZ::CC) && shortenOn001(MI, Opcode)) {
 144:     MachineInstrBuilder(*MI.getParent()->getParent(), &MI)
```
- **EN**: The range implements or declares functions including `SystemZShortenInst::shortenOn001`, `SystemZShortenInst::shortenOn001AddCC`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZShortenInst::shortenOn001`, `SystemZShortenInst::shortenOn001AddCC` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 145-162 / 第 145-162 行
```cpp
 145:       .addReg(SystemZ::CC, RegState::ImplicitDefine | RegState::Dead);
 146:     return true;
 147:   }
 148:   return false;
 149: }
 150: 
 151: // MI is a vector-style conversion instruction with the operand order:
 152: // destination, source, exact-suppress, rounding-mode.  If both registers
 153: // have a 4-bit encoding then change it to Opcode, which has operand order:
 154: // destination, rouding-mode, source, exact-suppress.
 155: bool SystemZShortenInst::shortenFPConv(MachineInstr &MI, unsigned Opcode) {
 156:   if (SystemZMC::getFirstReg(MI.getOperand(0).getReg()) < 16 &&
 157:       SystemZMC::getFirstReg(MI.getOperand(1).getReg()) < 16) {
 158:     MachineOperand Dest(MI.getOperand(0));
 159:     MachineOperand Src(MI.getOperand(1));
 160:     MachineOperand Suppress(MI.getOperand(2));
 161:     MachineOperand Mode(MI.getOperand(3));
 162:     MI.removeOperand(3);
```
- **EN**: The range implements or declares functions including `SystemZShortenInst::shortenFPConv`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZShortenInst::shortenFPConv` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 163-180 / 第 163-180 行
```cpp
 163:     MI.removeOperand(2);
 164:     MI.removeOperand(1);
 165:     MI.removeOperand(0);
 166:     MI.setDesc(TII->get(Opcode));
 167:     MachineInstrBuilder(*MI.getParent()->getParent(), &MI)
 168:         .add(Dest)
 169:         .add(Mode)
 170:         .add(Src)
 171:         .add(Suppress);
 172:     return true;
 173:   }
 174:   return false;
 175: }
 176: 
 177: bool SystemZShortenInst::shortenFusedFPOp(MachineInstr &MI, unsigned Opcode) {
 178:   MachineOperand &DstMO = MI.getOperand(0);
 179:   MachineOperand &LHSMO = MI.getOperand(1);
 180:   MachineOperand &RHSMO = MI.getOperand(2);
```
- **EN**: The range implements or declares functions including `MachineInstrBuilder`, `SystemZShortenInst::shortenFusedFPOp`.
- **CN**: 这一段实现或声明了 `MachineInstrBuilder`, `SystemZShortenInst::shortenFusedFPOp` 等函数。

### Lines 181-198 / 第 181-198 行
```cpp
 181:   MachineOperand &AccMO = MI.getOperand(3);
 182:   if (SystemZMC::getFirstReg(DstMO.getReg()) < 16 &&
 183:       SystemZMC::getFirstReg(LHSMO.getReg()) < 16 &&
 184:       SystemZMC::getFirstReg(RHSMO.getReg()) < 16 &&
 185:       SystemZMC::getFirstReg(AccMO.getReg()) < 16 &&
 186:       DstMO.getReg() == AccMO.getReg()) {
 187:     MachineOperand Lhs(LHSMO);
 188:     MachineOperand Rhs(RHSMO);
 189:     MachineOperand Src(AccMO);
 190:     MI.removeOperand(3);
 191:     MI.removeOperand(2);
 192:     MI.removeOperand(1);
 193:     MI.setDesc(TII->get(Opcode));
 194:     MachineInstrBuilder(*MI.getParent()->getParent(), &MI)
 195:         .add(Src)
 196:         .add(Lhs)
 197:         .add(Rhs);
 198:     return true;
```
- **EN**: The range implements or declares functions including `MachineInstrBuilder`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `MachineInstrBuilder` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 199-216 / 第 199-216 行
```cpp
 199:   }
 200:   return false;
 201: }
 202: 
 203: // Process all instructions in MBB.  Return true if something changed.
 204: bool SystemZShortenInst::processBlock(MachineBasicBlock &MBB) {
 205:   bool Changed = false;
 206: 
 207:   // Set up the set of live registers at the end of MBB (live out)
 208:   LiveRegs.clear();
 209:   LiveRegs.addLiveOuts(MBB);
 210: 
 211:   // Iterate backwards through the block looking for instructions to change.
 212:   for (MachineInstr &MI : llvm::reverse(MBB)) {
 213:     switch (MI.getOpcode()) {
 214:     case SystemZ::IILF:
 215:       Changed |= shortenIIF(MI, SystemZ::LLILL, SystemZ::LLILH);
 216:       break;
```
- **EN**: The range implements or declares functions including `SystemZShortenInst::processBlock`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZShortenInst::processBlock` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 217-234 / 第 217-234 行
```cpp
 217: 
 218:     case SystemZ::IIHF:
 219:       Changed |= shortenIIF(MI, SystemZ::LLIHL, SystemZ::LLIHH);
 220:       break;
 221: 
 222:     case SystemZ::WFADB:
 223:       Changed |= shortenOn001AddCC(MI, SystemZ::ADBR);
 224:       break;
 225: 
 226:     case SystemZ::WFASB:
 227:       Changed |= shortenOn001AddCC(MI, SystemZ::AEBR);
 228:       break;
 229: 
 230:     case SystemZ::WFDDB:
 231:       Changed |= shortenOn001(MI, SystemZ::DDBR);
 232:       break;
 233: 
 234:     case SystemZ::WFDSB:
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 235-252 / 第 235-252 行
```cpp
 235:       Changed |= shortenOn001(MI, SystemZ::DEBR);
 236:       break;
 237: 
 238:     case SystemZ::WFIDB:
 239:       Changed |= shortenFPConv(MI, SystemZ::FIDBRA);
 240:       break;
 241: 
 242:     case SystemZ::WFISB:
 243:       Changed |= shortenFPConv(MI, SystemZ::FIEBRA);
 244:       break;
 245: 
 246:     case SystemZ::WLDEB:
 247:       Changed |= shortenOn01(MI, SystemZ::LDEBR);
 248:       break;
 249: 
 250:     case SystemZ::WLEDB:
 251:       Changed |= shortenFPConv(MI, SystemZ::LEDBRA);
 252:       break;
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 253-270 / 第 253-270 行
```cpp
 253: 
 254:     case SystemZ::WFMDB:
 255:       Changed |= shortenOn001(MI, SystemZ::MDBR);
 256:       break;
 257: 
 258:     case SystemZ::WFMSB:
 259:       Changed |= shortenOn001(MI, SystemZ::MEEBR);
 260:       break;
 261: 
 262:     case SystemZ::WFMADB:
 263:       Changed |= shortenFusedFPOp(MI, SystemZ::MADBR);
 264:       break;
 265: 
 266:     case SystemZ::WFMASB:
 267:       Changed |= shortenFusedFPOp(MI, SystemZ::MAEBR);
 268:       break;
 269: 
 270:     case SystemZ::WFMSDB:
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 271-288 / 第 271-288 行
```cpp
 271:       Changed |= shortenFusedFPOp(MI, SystemZ::MSDBR);
 272:       break;
 273: 
 274:     case SystemZ::WFMSSB:
 275:       Changed |= shortenFusedFPOp(MI, SystemZ::MSEBR);
 276:       break;
 277: 
 278:     case SystemZ::WFLCDB:
 279:       Changed |= shortenOn01(MI, SystemZ::LCDFR);
 280:       break;
 281: 
 282:     case SystemZ::WFLCSB:
 283:       Changed |= shortenOn01(MI, SystemZ::LCDFR_32);
 284:       break;
 285: 
 286:     case SystemZ::WFLNDB:
 287:       Changed |= shortenOn01(MI, SystemZ::LNDFR);
 288:       break;
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 289-306 / 第 289-306 行
```cpp
 289: 
 290:     case SystemZ::WFLNSB:
 291:       Changed |= shortenOn01(MI, SystemZ::LNDFR_32);
 292:       break;
 293: 
 294:     case SystemZ::WFLPDB:
 295:       Changed |= shortenOn01(MI, SystemZ::LPDFR);
 296:       break;
 297: 
 298:     case SystemZ::WFLPSB:
 299:       Changed |= shortenOn01(MI, SystemZ::LPDFR_32);
 300:       break;
 301: 
 302:     case SystemZ::WFSQDB:
 303:       Changed |= shortenOn01(MI, SystemZ::SQDBR);
 304:       break;
 305: 
 306:     case SystemZ::WFSQSB:
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 307-324 / 第 307-324 行
```cpp
 307:       Changed |= shortenOn01(MI, SystemZ::SQEBR);
 308:       break;
 309: 
 310:     case SystemZ::WFSDB:
 311:       Changed |= shortenOn001AddCC(MI, SystemZ::SDBR);
 312:       break;
 313: 
 314:     case SystemZ::WFSSB:
 315:       Changed |= shortenOn001AddCC(MI, SystemZ::SEBR);
 316:       break;
 317: 
 318:     case SystemZ::WFCDB:
 319:       Changed |= shortenOn01(MI, SystemZ::CDBR);
 320:       break;
 321: 
 322:     case SystemZ::WFCSB:
 323:       Changed |= shortenOn01(MI, SystemZ::CEBR);
 324:       break;
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 325-342 / 第 325-342 行
```cpp
 325: 
 326:     case SystemZ::WFKDB:
 327:       Changed |= shortenOn01(MI, SystemZ::KDBR);
 328:       break;
 329: 
 330:     case SystemZ::WFKSB:
 331:       Changed |= shortenOn01(MI, SystemZ::KEBR);
 332:       break;
 333: 
 334:     case SystemZ::VL32:
 335:       // For z13 we prefer LDE over LE to avoid partial register dependencies.
 336:       Changed |= shortenOn0(MI, SystemZ::LDE32);
 337:       break;
 338: 
 339:     case SystemZ::VST32:
 340:       Changed |= shortenOn0(MI, SystemZ::STE);
 341:       break;
 342: 
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 343-360 / 第 343-360 行
```cpp
 343:     case SystemZ::VL64:
 344:       Changed |= shortenOn0(MI, SystemZ::LD);
 345:       break;
 346: 
 347:     case SystemZ::VST64:
 348:       Changed |= shortenOn0(MI, SystemZ::STD);
 349:       break;
 350: 
 351:     default: {
 352:       int TwoOperandOpcode = SystemZ::getTwoOperandOpcode(MI.getOpcode());
 353:       if (TwoOperandOpcode == -1)
 354:         break;
 355: 
 356:       if ((MI.getOperand(0).getReg() != MI.getOperand(1).getReg()) &&
 357:           (!MI.isCommutable() ||
 358:            MI.getOperand(0).getReg() != MI.getOperand(2).getReg() ||
 359:            !TII->commuteInstruction(MI, false, 1, 2)))
 360:           break;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 361-378 / 第 361-378 行
```cpp
 361: 
 362:       MI.setDesc(TII->get(TwoOperandOpcode));
 363:       MI.tieOperands(0, 1);
 364:       if (TwoOperandOpcode == SystemZ::SLL ||
 365:           TwoOperandOpcode == SystemZ::SLA ||
 366:           TwoOperandOpcode == SystemZ::SRL ||
 367:           TwoOperandOpcode == SystemZ::SRA) {
 368:         // These shifts only use the low 6 bits of the shift count.
 369:         MachineOperand &ImmMO = MI.getOperand(3);
 370:         ImmMO.setImm(ImmMO.getImm() & 0xfff);
 371:       }
 372:       Changed = true;
 373:       break;
 374:     }
 375:     }
 376:     if (!MI.isDebugInstr())
 377:       LiveRegs.stepBackward(MI);
 378:   }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 379-396 / 第 379-396 行
```cpp
 379: 
 380:   return Changed;
 381: }
 382: 
 383: bool SystemZShortenInst::runOnMachineFunction(MachineFunction &F) {
 384:   if (skipFunction(F.getFunction()))
 385:     return false;
 386: 
 387:   const SystemZSubtarget &ST = F.getSubtarget<SystemZSubtarget>();
 388:   TII = ST.getInstrInfo();
 389:   TRI = ST.getRegisterInfo();
 390:   LiveRegs.init(*TRI);
 391: 
 392:   bool Changed = false;
 393:   for (auto &MBB : F)
 394:     Changed |= processBlock(MBB);
 395: 
 396:   return Changed;
```
- **EN**: The range implements or declares functions including `SystemZShortenInst::runOnMachineFunction`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZShortenInst::runOnMachineFunction` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 397-397 / 第 397-397 行
```cpp
 397: }
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Register classes**: Groups registers by capability so instruction operands and allocators can constrain them. / 按能力对寄存器分组，以便指令操作数和分配器施加约束。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `SystemZTargetMachine.h`
- `llvm/CodeGen/LiveRegUnits.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/TargetRegisterInfo.h`
