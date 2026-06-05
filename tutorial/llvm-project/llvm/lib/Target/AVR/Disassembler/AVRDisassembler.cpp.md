# AVRDisassembler.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/AVR/Disassembler/AVRDisassembler.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file is part of the AVR Disassembler.
- 目的（中文）: 实现目标反汇编器，将二进制指令解码回 MC 指令。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===- AVRDisassembler.cpp - Disassembler for AVR ---------------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file is part of the AVR Disassembler.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "MCTargetDesc/AVRMCTargetDesc.h"
  14: #include "TargetInfo/AVRTargetInfo.h"
  15: 
  16: #include "llvm/ADT/DenseMap.h"
  17: #include "llvm/ADT/STLExtras.h"
  18: 
  19: #include "llvm/MC/MCAsmInfo.h"
  20: #include "llvm/MC/MCContext.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 21-40

```cpp
  21: #include "llvm/MC/MCDecoder.h"
  22: #include "llvm/MC/MCDecoderOps.h"
  23: #include "llvm/MC/MCDisassembler/MCDisassembler.h"
  24: #include "llvm/MC/MCInst.h"
  25: #include "llvm/MC/TargetRegistry.h"
  26: 
  27: #include "llvm/Support/Compiler.h"
  28: 
  29: using namespace llvm;
  30: using namespace llvm::MCD;
  31: 
  32: #define DEBUG_TYPE "avr-disassembler"
  33: 
  34: typedef MCDisassembler::DecodeStatus DecodeStatus;
  35: 
  36: namespace {
  37: 
  38: /// A disassembler class for AVR.
  39: class AVRDisassembler : public MCDisassembler {
  40: public:
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as for, AVRDisassembler, which organize the target-specific behavior exposed by the file. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 for, AVRDisassembler 等接口或数据结构，用于组织该文件暴露的目标专用行为。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 41-60

```cpp
  41:   AVRDisassembler(const MCSubtargetInfo &STI, MCContext &Ctx)
  42:       : MCDisassembler(STI, Ctx) {}
  43:   ~AVRDisassembler() override = default;
  44: 
  45:   DecodeStatus getInstruction(MCInst &Instr, uint64_t &Size,
  46:                               ArrayRef<uint8_t> Bytes, uint64_t Address,
  47:                               raw_ostream &CStream) const override;
  48: };
  49: } // namespace
  50: 
  51: static MCDisassembler *createAVRDisassembler(const Target &T,
  52:                                              const MCSubtargetInfo &STI,
  53:                                              MCContext &Ctx) {
  54:   return new AVRDisassembler(STI, Ctx);
  55: }
  56: 
  57: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
  58: LLVMInitializeAVRDisassembler() {
  59:   // Register the disassembler.
  60:   TargetRegistry::RegisterMCDisassembler(getTheAVRTarget(),
```

- EN: Function bodies or method definitions such as MCDisassembler contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: MCDisassembler 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 61-80

```cpp
  61:                                          createAVRDisassembler);
  62: }
  63: 
  64: static constexpr MCRegister GPRDecoderTable[] = {
  65:     AVR::R0,  AVR::R1,  AVR::R2,  AVR::R3,  AVR::R4,  AVR::R5,  AVR::R6,
  66:     AVR::R7,  AVR::R8,  AVR::R9,  AVR::R10, AVR::R11, AVR::R12, AVR::R13,
  67:     AVR::R14, AVR::R15, AVR::R16, AVR::R17, AVR::R18, AVR::R19, AVR::R20,
  68:     AVR::R21, AVR::R22, AVR::R23, AVR::R24, AVR::R25, AVR::R26, AVR::R27,
  69:     AVR::R28, AVR::R29, AVR::R30, AVR::R31,
  70: };
  71: 
  72: static constexpr MCRegister GPRPairDecoderTable[] = {
  73:     AVR::R1R0,   AVR::R3R2,   AVR::R5R4,   AVR::R7R6,
  74:     AVR::R9R8,   AVR::R11R10, AVR::R13R12, AVR::R15R14,
  75:     AVR::R17R16, AVR::R19R18, AVR::R21R20, AVR::R23R22,
  76:     AVR::R25R24, AVR::R27R26, AVR::R29R28, AVR::R31R30,
  77: };
  78: 
  79: static DecodeStatus DecodeGPR8RegisterClass(MCInst &Inst, unsigned RegNo,
  80:                                             uint64_t Address,
```

- EN: In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 81-100

```cpp
  81:                                             const MCDisassembler *Decoder) {
  82:   if (RegNo > 31)
  83:     return MCDisassembler::Fail;
  84: 
  85:   MCRegister Register = GPRDecoderTable[RegNo];
  86:   Inst.addOperand(MCOperand::createReg(Register));
  87:   return MCDisassembler::Success;
  88: }
  89: 
  90: static DecodeStatus DecodeLD8RegisterClass(MCInst &Inst, unsigned RegNo,
  91:                                            uint64_t Address,
  92:                                            const MCDisassembler *Decoder) {
  93:   assert(isUInt<4>(RegNo));
  94:   // Only r16...r31 are legal.
  95:   Inst.addOperand(MCOperand::createReg(GPRDecoderTable[16 + RegNo]));
  96:   return MCDisassembler::Success;
  97: }
  98: 
  99: static DecodeStatus DecodeLD8loRegisterClass(MCInst &Inst, unsigned RegNo,
 100:                                              uint64_t Address,
```

- EN: Function bodies or method definitions such as DecodeLD8RegisterClass contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodeLD8RegisterClass 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 101-120

```cpp
 101:                                              const MCDisassembler *Decoder) {
 102:   assert(isUInt<3>(RegNo));
 103:   // Only r16...r23 are legal.
 104:   Inst.addOperand(MCOperand::createReg(GPRDecoderTable[16 + RegNo]));
 105:   return MCDisassembler::Success;
 106: }
 107: 
 108: static DecodeStatus DecodeDREGSRegisterClass(MCInst &Inst, unsigned RegNo,
 109:                                              uint64_t Address,
 110:                                              const MCDisassembler *Decoder) {
 111:   assert(isUInt<4>(RegNo));
 112:   Inst.addOperand(MCOperand::createReg(GPRPairDecoderTable[RegNo]));
 113:   return MCDisassembler::Success;
 114: }
 115: 
 116: static DecodeStatus DecodeIWREGSRegisterClass(MCInst &Inst, unsigned RegNo,
 117:                                               uint64_t Address,
 118:                                               const MCDisassembler *Decoder) {
 119:   assert(isUInt<2>(RegNo));
 120:   // Only AVR::R25R24, AVR::R27R26, AVR::R29R28, AVR::R31R30 are legal.
```

- EN: Function bodies or method definitions such as DecodeDREGSRegisterClass, DecodeIWREGSRegisterClass contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodeDREGSRegisterClass, DecodeIWREGSRegisterClass 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 121-140

```cpp
 121:   Inst.addOperand(MCOperand::createReg(GPRPairDecoderTable[12 + RegNo]));
 122:   return MCDisassembler::Success;
 123: }
 124: 
 125: static DecodeStatus DecodeZREGRegisterClass(MCInst &Inst,
 126:                                             const MCDisassembler *Decoder) {
 127:   Inst.addOperand(MCOperand::createReg(AVR::R31R30));
 128:   return MCDisassembler::Success;
 129: }
 130: 
 131: static DecodeStatus decodeCallTarget(MCInst &Inst, unsigned Field,
 132:                                      uint64_t Address,
 133:                                      const MCDisassembler *Decoder) {
 134:   // Call targets need to be shifted left by one so this needs a custom
 135:   // decoder.
 136:   Inst.addOperand(MCOperand::createImm(Field << 1));
 137:   return MCDisassembler::Success;
 138: }
 139: 
 140: static DecodeStatus decodeRelCondBrTarget7(MCInst &Inst, unsigned Field,
```

- EN: Function bodies or method definitions such as DecodeZREGRegisterClass, decodeCallTarget contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodeZREGRegisterClass, decodeCallTarget 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 141-160

```cpp
 141:                                            uint64_t Address,
 142:                                            const MCDisassembler *Decoder) {
 143:   // The legal range is [-128, 126] (in bytes).
 144:   Inst.addOperand(MCOperand::createImm(SignExtend32(Field, 7) * 2));
 145:   return MCDisassembler::Success;
 146: }
 147: 
 148: static DecodeStatus decodeRelCondBrTarget13(MCInst &Inst, unsigned Field,
 149:                                             uint64_t Address,
 150:                                             const MCDisassembler *Decoder) {
 151:   // The legal range is [-4096, 4094] (in bytes).
 152:   Inst.addOperand(MCOperand::createImm(SignExtend32(Field, 12) * 2));
 153:   return MCDisassembler::Success;
 154: }
 155: 
 156: static DecodeStatus decodeMemri(MCInst &Inst, unsigned Insn, uint64_t Address,
 157:                                 const MCDisassembler *Decoder) {
 158:   // As in the EncoderMethod `AVRMCCodeEmitter::encodeMemri`, the memory
 159:   // address is encoded into 7-bit, in which bits 0-5 are the immediate offset,
 160:   // and the bit-6 is the pointer register bit (Z=0, Y=1).
```

- EN: Function bodies or method definitions such as decodeRelCondBrTarget13, decodeMemri contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: decodeRelCondBrTarget13, decodeMemri 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 161-180

```cpp
 161:   if (Insn > 127)
 162:     return MCDisassembler::Fail;
 163: 
 164:   // Append the base register operand.
 165:   Inst.addOperand(
 166:       MCOperand::createReg((Insn & 0x40) ? AVR::R29R28 : AVR::R31R30));
 167:   // Append the immediate offset operand.
 168:   Inst.addOperand(MCOperand::createImm(Insn & 0x3f));
 169: 
 170:   return MCDisassembler::Success;
 171: }
 172: 
 173: static DecodeStatus decodeLoadStore(MCInst &Inst, unsigned Insn,
 174:                                     uint64_t Address,
 175:                                     const MCDisassembler *Decoder) {
 176:   // Get the register will be loaded or stored.
 177:   MCRegister RegVal = GPRDecoderTable[(Insn >> 4) & 0x1f];
 178: 
 179:   // Decode LDD/STD with offset less than 8.
 180:   if ((Insn & 0xf000) == 0x8000) {
```

- EN: Function bodies or method definitions such as decodeLoadStore contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: decodeLoadStore 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 181-200

```cpp
 181:     unsigned RegBase = (Insn & 0x8) ? AVR::R29R28 : AVR::R31R30;
 182:     unsigned Offset = Insn & 7; // We need not consider offset > 7.
 183:     if ((Insn & 0x200) == 0) {  // Decode LDD.
 184:       Inst.setOpcode(AVR::LDDRdPtrQ);
 185:       Inst.addOperand(MCOperand::createReg(RegVal));
 186:       Inst.addOperand(MCOperand::createReg(RegBase));
 187:       Inst.addOperand(MCOperand::createImm(Offset));
 188:     } else { // Decode STD.
 189:       Inst.setOpcode(AVR::STDPtrQRr);
 190:       Inst.addOperand(MCOperand::createReg(RegBase));
 191:       Inst.addOperand(MCOperand::createImm(Offset));
 192:       Inst.addOperand(MCOperand::createReg(RegVal));
 193:     }
 194:     return MCDisassembler::Success;
 195:   }
 196: 
 197:   // Decode the following 14 instructions. Bit 9 indicates load(0) or store(1),
 198:   // bits 8~4 indicate the value register, bits 3-2 indicate the base address
 199:   // register (11-X, 10-Y, 00-Z), bits 1~0 indicate the mode (00-basic,
 200:   // 01-postinc, 10-predec).
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 201-220

```cpp
 201:   // ST X,  Rr : 1001 001r rrrr 1100
 202:   // ST X+, Rr : 1001 001r rrrr 1101
 203:   // ST -X, Rr : 1001 001r rrrr 1110
 204:   // ST Y+, Rr : 1001 001r rrrr 1001
 205:   // ST -Y, Rr : 1001 001r rrrr 1010
 206:   // ST Z+, Rr : 1001 001r rrrr 0001
 207:   // ST -Z, Rr : 1001 001r rrrr 0010
 208:   // LD Rd, X  : 1001 000d dddd 1100
 209:   // LD Rd, X+ : 1001 000d dddd 1101
 210:   // LD Rd, -X : 1001 000d dddd 1110
 211:   // LD Rd, Y+ : 1001 000d dddd 1001
 212:   // LD Rd, -Y : 1001 000d dddd 1010
 213:   // LD Rd, Z+ : 1001 000d dddd 0001
 214:   // LD Rd, -Z : 1001 000d dddd 0010
 215:   if ((Insn & 0xfc00) != 0x9000 || (Insn & 0xf) == 0)
 216:     return MCDisassembler::Fail;
 217: 
 218:   // Get the base address register.
 219:   unsigned RegBase;
 220:   switch (Insn & 0xc) {
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 221-240

```cpp
 221:   case 0xc:
 222:     RegBase = AVR::R27R26;
 223:     break;
 224:   case 0x8:
 225:     RegBase = AVR::R29R28;
 226:     break;
 227:   case 0x0:
 228:     RegBase = AVR::R31R30;
 229:     break;
 230:   default:
 231:     return MCDisassembler::Fail;
 232:   }
 233: 
 234:   // Set the opcode.
 235:   switch (Insn & 0x203) {
 236:   case 0x200:
 237:     Inst.setOpcode(AVR::STPtrRr);
 238:     Inst.addOperand(MCOperand::createReg(RegBase));
 239:     Inst.addOperand(MCOperand::createReg(RegVal));
 240:     return MCDisassembler::Success;
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 241-260

```cpp
 241:   case 0x201:
 242:     Inst.setOpcode(AVR::STPtrPiRr);
 243:     break;
 244:   case 0x202:
 245:     Inst.setOpcode(AVR::STPtrPdRr);
 246:     break;
 247:   case 0:
 248:     Inst.setOpcode(AVR::LDRdPtr);
 249:     Inst.addOperand(MCOperand::createReg(RegVal));
 250:     Inst.addOperand(MCOperand::createReg(RegBase));
 251:     return MCDisassembler::Success;
 252:   case 1:
 253:     Inst.setOpcode(AVR::LDRdPtrPi);
 254:     break;
 255:   case 2:
 256:     Inst.setOpcode(AVR::LDRdPtrPd);
 257:     break;
 258:   default:
 259:     return MCDisassembler::Fail;
 260:   }
```

- EN: In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 261-280

```cpp
 261: 
 262:   // Build postinc/predec machine instructions.
 263:   if ((Insn & 0x200) == 0) { // This is a load instruction.
 264:     Inst.addOperand(MCOperand::createReg(RegVal));
 265:     Inst.addOperand(MCOperand::createReg(RegBase));
 266:     Inst.addOperand(MCOperand::createReg(RegBase));
 267:   } else { // This is a store instruction.
 268:     Inst.addOperand(MCOperand::createReg(RegBase));
 269:     Inst.addOperand(MCOperand::createReg(RegBase));
 270:     Inst.addOperand(MCOperand::createReg(RegVal));
 271:     // STPtrPiRr and STPtrPdRr have an extra immediate operand.
 272:     Inst.addOperand(MCOperand::createImm(1));
 273:   }
 274: 
 275:   return MCDisassembler::Success;
 276: }
 277: 
 278: #include "AVRGenDisassemblerTables.inc"
 279: 
 280: static DecodeStatus readInstruction16(ArrayRef<uint8_t> Bytes, uint64_t Address,
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 281-300

```cpp
 281:                                       uint64_t &Size, uint32_t &Insn) {
 282:   if (Bytes.size() < 2) {
 283:     Size = 0;
 284:     return MCDisassembler::Fail;
 285:   }
 286: 
 287:   Size = 2;
 288:   Insn = (Bytes[0] << 0) | (Bytes[1] << 8);
 289: 
 290:   return MCDisassembler::Success;
 291: }
 292: 
 293: static DecodeStatus readInstruction32(ArrayRef<uint8_t> Bytes, uint64_t Address,
 294:                                       uint64_t &Size, uint32_t &Insn) {
 295: 
 296:   if (Bytes.size() < 4) {
 297:     Size = 0;
 298:     return MCDisassembler::Fail;
 299:   }
 300: 
```

- EN: Function bodies or method definitions such as readInstruction32 contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: readInstruction32 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 301-320

```cpp
 301:   Size = 4;
 302:   Insn =
 303:       (Bytes[0] << 16) | (Bytes[1] << 24) | (Bytes[2] << 0) | (Bytes[3] << 8);
 304: 
 305:   return MCDisassembler::Success;
 306: }
 307: 
 308: static const uint8_t *getDecoderTable(uint64_t Size) {
 309: 
 310:   switch (Size) {
 311:   case 2:
 312:     return DecoderTable16;
 313:   case 4:
 314:     return DecoderTable32;
 315:   default:
 316:     llvm_unreachable("instructions must be 16 or 32-bits");
 317:   }
 318: }
 319: 
 320: DecodeStatus AVRDisassembler::getInstruction(MCInst &Instr, uint64_t &Size,
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 321-340

```cpp
 321:                                              ArrayRef<uint8_t> Bytes,
 322:                                              uint64_t Address,
 323:                                              raw_ostream &CStream) const {
 324:   uint32_t Insn;
 325: 
 326:   DecodeStatus Result;
 327: 
 328:   // Try decode a 16-bit instruction.
 329:   {
 330:     Result = readInstruction16(Bytes, Address, Size, Insn);
 331: 
 332:     if (Result == MCDisassembler::Fail)
 333:       return MCDisassembler::Fail;
 334: 
 335:     // Try to decode AVRTiny instructions.
 336:     if (STI.hasFeature(AVR::FeatureTinyEncoding)) {
 337:       Result = decodeInstruction(DecoderTableAVRTiny16, Instr, Insn, Address,
 338:                                  this, STI);
 339:       if (Result != MCDisassembler::Fail)
 340:         return Result;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 341-360

```cpp
 341:     }
 342: 
 343:     // Try to auto-decode a 16-bit instruction.
 344:     Result = decodeInstruction(getDecoderTable(Size), Instr, Insn, Address,
 345:                                this, STI);
 346:     if (Result != MCDisassembler::Fail)
 347:       return Result;
 348: 
 349:     // Try to decode to a load/store instruction. ST/LD need a specified
 350:     // DecoderMethod, as they already have a specified PostEncoderMethod.
 351:     Result = decodeLoadStore(Instr, Insn, Address, this);
 352:     if (Result != MCDisassembler::Fail)
 353:       return Result;
 354:   }
 355: 
 356:   // Try decode a 32-bit instruction.
 357:   {
 358:     Result = readInstruction32(Bytes, Address, Size, Insn);
 359: 
 360:     if (Result == MCDisassembler::Fail)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 361-375

```cpp
 361:       return MCDisassembler::Fail;
 362: 
 363:     Result = decodeInstruction(getDecoderTable(Size), Instr, Insn, Address,
 364:                                this, STI);
 365: 
 366:     if (Result != MCDisassembler::Fail) {
 367:       return Result;
 368:     }
 369: 
 370:     return MCDisassembler::Fail;
 371:   }
 372: }
 373: 
 374: typedef DecodeStatus (*DecodeFunc)(MCInst &MI, unsigned insn, uint64_t Address,
 375:                                    const MCDisassembler *Decoder);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

## Key Concepts / 关键概念

- Decode tables / 解码表
- MCInst reconstruction / MCInst 重建
- MC instruction representation / MC 指令表示
- Binary encoding / 二进制编码
- Register classes / 寄存器类
- CPU feature modelling / CPU 特性建模
- Instruction decoding / 指令解码

## Dependencies / 依赖关系

- Direct includes / 直接包含: `MCTargetDesc/AVRMCTargetDesc.h`, `TargetInfo/AVRTargetInfo.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/STLExtras.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCDecoder.h`, `llvm/MC/MCDecoderOps.h`
- LLVM subsystems / LLVM 子系统: LLVM MC
- Generated or companion files / 生成或配套文件: `AVRGenDisassemblerTables.inc`
