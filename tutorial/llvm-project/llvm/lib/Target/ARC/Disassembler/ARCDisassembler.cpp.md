# ARCDisassembler.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/ARC/Disassembler/ARCDisassembler.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file is part of the ARC Disassembler.
- 目的（中文）: 实现目标反汇编器，将二进制指令解码回 MC 指令。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===- ARCDisassembler.cpp - Disassembler for ARC ---------------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// This file is part of the ARC Disassembler.
  11: ///
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "ARC.h"
  15: #include "ARCRegisterInfo.h"
  16: #include "MCTargetDesc/ARCMCTargetDesc.h"
  17: #include "TargetInfo/ARCTargetInfo.h"
  18: #include "llvm/MC/MCContext.h"
  19: #include "llvm/MC/MCDecoder.h"
  20: #include "llvm/MC/MCDecoderOps.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 21-40

```cpp
  21: #include "llvm/MC/MCDisassembler/MCDisassembler.h"
  22: #include "llvm/MC/MCInst.h"
  23: #include "llvm/MC/MCInstrInfo.h"
  24: #include "llvm/MC/MCSubtargetInfo.h"
  25: #include "llvm/MC/TargetRegistry.h"
  26: 
  27: using namespace llvm;
  28: using namespace llvm::MCD;
  29: 
  30: #define DEBUG_TYPE "arc-disassembler"
  31: 
  32: using DecodeStatus = MCDisassembler::DecodeStatus;
  33: 
  34: namespace {
  35: 
  36: /// A disassembler class for ARC.
  37: class ARCDisassembler : public MCDisassembler {
  38: public:
  39:   std::unique_ptr<MCInstrInfo const> const MCII;
  40: 
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as for, ARCDisassembler, which organize the target-specific behavior exposed by the file. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 for, ARCDisassembler 等接口或数据结构，用于组织该文件暴露的目标专用行为。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 41-60

```cpp
  41:   ARCDisassembler(const MCSubtargetInfo &STI, MCContext &Ctx,
  42:                   MCInstrInfo const *MCII)
  43:       : MCDisassembler(STI, Ctx), MCII(MCII) {}
  44: 
  45:   DecodeStatus getInstruction(MCInst &Instr, uint64_t &Size,
  46:                               ArrayRef<uint8_t> Bytes, uint64_t Address,
  47:                               raw_ostream &CStream) const override;
  48: };
  49: 
  50: } // end anonymous namespace
  51: 
  52: static bool readInstruction32(ArrayRef<uint8_t> Bytes, uint64_t Address,
  53:                               uint64_t &Size, uint32_t &Insn) {
  54:   Size = 4;
  55:   // Read 2 16-bit values, but swap hi/lo parts.
  56:   Insn =
  57:       (Bytes[0] << 16) | (Bytes[1] << 24) | (Bytes[2] << 0) | (Bytes[3] << 8);
  58:   return true;
  59: }
  60: 
```

- EN: Function bodies or method definitions such as MCDisassembler, readInstruction32 contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: MCDisassembler, readInstruction32 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 61-80

```cpp
  61: static bool readInstruction64(ArrayRef<uint8_t> Bytes, uint64_t Address,
  62:                               uint64_t &Size, uint64_t &Insn) {
  63:   Size = 8;
  64:   Insn = ((uint64_t)Bytes[0] << 16) | ((uint64_t)Bytes[1] << 24) |
  65:          ((uint64_t)Bytes[2] << 0) | ((uint64_t)Bytes[3] << 8) |
  66:          ((uint64_t)Bytes[4] << 48) | ((uint64_t)Bytes[5] << 56) |
  67:          ((uint64_t)Bytes[6] << 32) | ((uint64_t)Bytes[7] << 40);
  68:   return true;
  69: }
  70: 
  71: static bool readInstruction48(ArrayRef<uint8_t> Bytes, uint64_t Address,
  72:                               uint64_t &Size, uint64_t &Insn) {
  73:   Size = 6;
  74:   Insn = ((uint64_t)Bytes[0] << 0) | ((uint64_t)Bytes[1] << 8) |
  75:          ((uint64_t)Bytes[2] << 32) | ((uint64_t)Bytes[3] << 40) |
  76:          ((uint64_t)Bytes[4] << 16) | ((uint64_t)Bytes[5] << 24);
  77:   return true;
  78: }
  79: 
  80: static bool readInstruction16(ArrayRef<uint8_t> Bytes, uint64_t Address,
```

- EN: Function bodies or method definitions such as readInstruction64, readInstruction48 contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: readInstruction64, readInstruction48 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 81-100

```cpp
  81:                               uint64_t &Size, uint32_t &Insn) {
  82:   Size = 2;
  83:   Insn = (Bytes[0] << 0) | (Bytes[1] << 8);
  84:   return true;
  85: }
  86: 
  87: template <unsigned B>
  88: static DecodeStatus
  89: DecodeSignedOperand(MCInst &Inst, unsigned InsnS, uint64_t Address = 0,
  90:                     const MCDisassembler *Decoder = nullptr);
  91: 
  92: template <unsigned B>
  93: static DecodeStatus
  94: DecodeFromCyclicRange(MCInst &Inst, unsigned InsnS, uint64_t Address = 0,
  95:                       const MCDisassembler *Decoder = nullptr);
  96: 
  97: template <unsigned B>
  98: static DecodeStatus DecodeBranchTargetS(MCInst &Inst, unsigned InsnS,
  99:                                         uint64_t Address,
 100:                                         const MCDisassembler *Decoder);
```

- EN: In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 101-120

```cpp
 101: 
 102: static DecodeStatus DecodeMEMrs9(MCInst &, unsigned, uint64_t,
 103:                                  const MCDisassembler *);
 104: 
 105: static DecodeStatus DecodeLdLImmInstruction(MCInst &, uint64_t, uint64_t,
 106:                                             const MCDisassembler *);
 107: 
 108: static DecodeStatus DecodeStLImmInstruction(MCInst &, uint64_t, uint64_t,
 109:                                             const MCDisassembler *);
 110: 
 111: static DecodeStatus DecodeLdRLImmInstruction(MCInst &, uint64_t, uint64_t,
 112:                                              const MCDisassembler *);
 113: 
 114: static DecodeStatus DecodeSOPwithRS12(MCInst &, uint64_t, uint64_t,
 115:                                       const MCDisassembler *);
 116: 
 117: static DecodeStatus DecodeSOPwithRU6(MCInst &, uint64_t, uint64_t,
 118:                                      const MCDisassembler *);
 119: 
 120: static DecodeStatus DecodeCCRU6Instruction(MCInst &, uint64_t, uint64_t,
```

- EN: In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 121-140

```cpp
 121:                                            const MCDisassembler *);
 122: 
 123: static DecodeStatus DecodeMoveHRegInstruction(MCInst &Inst, uint64_t, uint64_t,
 124:                                               const MCDisassembler *);
 125: 
 126: static const uint16_t GPR32DecoderTable[] = {
 127:     ARC::R0,  ARC::R1,    ARC::R2,  ARC::R3,   ARC::R4,  ARC::R5,  ARC::R6,
 128:     ARC::R7,  ARC::R8,    ARC::R9,  ARC::R10,  ARC::R11, ARC::R12, ARC::R13,
 129:     ARC::R14, ARC::R15,   ARC::R16, ARC::R17,  ARC::R18, ARC::R19, ARC::R20,
 130:     ARC::R21, ARC::R22,   ARC::R23, ARC::R24,  ARC::R25, ARC::GP,  ARC::FP,
 131:     ARC::SP,  ARC::ILINK, ARC::R30, ARC::BLINK};
 132: 
 133: static DecodeStatus DecodeGPR32RegisterClass(MCInst &Inst, unsigned RegNo,
 134:                                              uint64_t Address,
 135:                                              const MCDisassembler *Decoder) {
 136:   if (RegNo >= 32) {
 137:     LLVM_DEBUG(dbgs() << "Not a GPR32 register.");
 138:     return MCDisassembler::Fail;
 139:   }
 140: 
```

- EN: Function bodies or method definitions such as DecodeGPR32RegisterClass contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodeGPR32RegisterClass 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 141-160

```cpp
 141:   unsigned Reg = GPR32DecoderTable[RegNo];
 142:   Inst.addOperand(MCOperand::createReg(Reg));
 143:   return MCDisassembler::Success;
 144: }
 145: 
 146: static DecodeStatus DecodeGBR32ShortRegister(MCInst &Inst, unsigned RegNo,
 147:                                              uint64_t Address,
 148:                                              const MCDisassembler *Decoder) {
 149:   // Enumerates registers from ranges [r0-r3],[r12-r15].
 150:   if (RegNo > 3)
 151:     RegNo += 8; // 4 for r12, etc...
 152: 
 153:   return DecodeGPR32RegisterClass(Inst, RegNo, Address, Decoder);
 154: }
 155: 
 156: #include "ARCGenDisassemblerTables.inc"
 157: 
 158: static unsigned decodeCField(unsigned Insn) {
 159:   return fieldFromInstruction(Insn, 6, 6);
 160: }
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. Function bodies or method definitions such as DecodeGBR32ShortRegister, decodeCField contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 DecodeGBR32ShortRegister, decodeCField 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 161-180

```cpp
 161: 
 162: static unsigned decodeBField(unsigned Insn) {
 163:   return (fieldFromInstruction(Insn, 12, 3) << 3) |
 164:          fieldFromInstruction(Insn, 24, 3);
 165: }
 166: 
 167: static unsigned decodeAField(unsigned Insn) {
 168:   return fieldFromInstruction(Insn, 0, 6);
 169: }
 170: 
 171: static DecodeStatus DecodeMEMrs9(MCInst &Inst, unsigned Insn, uint64_t Address,
 172:                                  const MCDisassembler *Dec) {
 173:   // We have the 9-bit immediate in the low bits, 6-bit register in high bits.
 174:   unsigned S9 = Insn & 0x1ff;
 175:   unsigned R = (Insn & (0x7fff & ~0x1ff)) >> 9;
 176:   DecodeGPR32RegisterClass(Inst, R, Address, Dec);
 177:   Inst.addOperand(MCOperand::createImm(SignExtend32<9>(S9)));
 178:   return MCDisassembler::Success;
 179: }
 180: 
```

- EN: Function bodies or method definitions such as decodeBField, decodeAField, DecodeMEMrs9 contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: decodeBField, decodeAField, DecodeMEMrs9 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 181-200

```cpp
 181: static bool DecodeSymbolicOperand(MCInst &Inst, uint64_t Address,
 182:                                   uint64_t Value,
 183:                                   const MCDisassembler *Decoder) {
 184:   static const uint64_t AtLeast = 2;
 185:   return (nullptr != Decoder && Decoder->tryAddingSymbolicOperand(
 186:                                     Inst, Value, Address, true, 0, AtLeast, 0));
 187: }
 188: 
 189: static void DecodeSymbolicOperandOff(MCInst &Inst, uint64_t Address,
 190:                                      uint64_t Offset,
 191:                                      const MCDisassembler *Decoder) {
 192:   uint64_t NextAddress = Address + Offset;
 193: 
 194:   if (!DecodeSymbolicOperand(Inst, Address, NextAddress, Decoder))
 195:     Inst.addOperand(MCOperand::createImm(Offset));
 196: }
 197: 
 198: template <unsigned B>
 199: static DecodeStatus DecodeBranchTargetS(MCInst &Inst, unsigned InsnS,
 200:                                         uint64_t Address,
```

- EN: Function bodies or method definitions such as DecodeSymbolicOperand, DecodeSymbolicOperandOff contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodeSymbolicOperand, DecodeSymbolicOperandOff 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 201-220

```cpp
 201:                                         const MCDisassembler *Decoder) {
 202: 
 203:   static_assert(B > 0, "field is empty");
 204:   DecodeSymbolicOperandOff(Inst, Address, SignExtend32<B>(InsnS), Decoder);
 205:   return MCDisassembler::Success;
 206: }
 207: 
 208: template <unsigned B>
 209: static DecodeStatus DecodeSignedOperand(MCInst &Inst, unsigned InsnS,
 210:                                         uint64_t /*Address*/,
 211:                                         const MCDisassembler * /*Decoder*/) {
 212: 
 213:   static_assert(B > 0, "field is empty");
 214:   Inst.addOperand(MCOperand::createImm(
 215:       SignExtend32<B>(maskTrailingOnes<decltype(InsnS)>(B) & InsnS)));
 216:   return MCDisassembler::Success;
 217: }
 218: 
 219: template <unsigned B>
 220: static DecodeStatus DecodeFromCyclicRange(MCInst &Inst, unsigned InsnS,
```

- EN: Function bodies or method definitions such as DecodeSignedOperand contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodeSignedOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 221-240

```cpp
 221:                                           uint64_t /*Address*/,
 222:                                           const MCDisassembler * /*Decoder*/) {
 223: 
 224:   static_assert(B > 0, "field is empty");
 225:   const unsigned max = (1u << B) - 1;
 226:   Inst.addOperand(
 227:       MCOperand::createImm(InsnS < max ? static_cast<int>(InsnS) : -1));
 228:   return MCDisassembler::Success;
 229: }
 230: 
 231: static DecodeStatus DecodeStLImmInstruction(MCInst &Inst, uint64_t Insn,
 232:                                             uint64_t Address,
 233:                                             const MCDisassembler *Decoder) {
 234:   unsigned SrcC, DstB, LImm;
 235:   DstB = decodeBField(Insn);
 236:   if (DstB != 62) {
 237:     LLVM_DEBUG(dbgs() << "Decoding StLImm found non-limm register.");
 238:     return MCDisassembler::Fail;
 239:   }
 240:   SrcC = decodeCField(Insn);
```

- EN: Function bodies or method definitions such as DecodeStLImmInstruction contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodeStLImmInstruction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 241-260

```cpp
 241:   DecodeGPR32RegisterClass(Inst, SrcC, Address, Decoder);
 242:   LImm = (Insn >> 32);
 243:   Inst.addOperand(MCOperand::createImm(LImm));
 244:   Inst.addOperand(MCOperand::createImm(0));
 245:   return MCDisassembler::Success;
 246: }
 247: 
 248: static DecodeStatus DecodeLdLImmInstruction(MCInst &Inst, uint64_t Insn,
 249:                                             uint64_t Address,
 250:                                             const MCDisassembler *Decoder) {
 251:   unsigned DstA, SrcB, LImm;
 252:   LLVM_DEBUG(dbgs() << "Decoding LdLImm:\n");
 253:   SrcB = decodeBField(Insn);
 254:   if (SrcB != 62) {
 255:     LLVM_DEBUG(dbgs() << "Decoding LdLImm found non-limm register.");
 256:     return MCDisassembler::Fail;
 257:   }
 258:   DstA = decodeAField(Insn);
 259:   DecodeGPR32RegisterClass(Inst, DstA, Address, Decoder);
 260:   LImm = (Insn >> 32);
```

- EN: Function bodies or method definitions such as DecodeLdLImmInstruction contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodeLdLImmInstruction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 261-280

```cpp
 261:   Inst.addOperand(MCOperand::createImm(LImm));
 262:   Inst.addOperand(MCOperand::createImm(0));
 263:   return MCDisassembler::Success;
 264: }
 265: 
 266: static DecodeStatus DecodeLdRLImmInstruction(MCInst &Inst, uint64_t Insn,
 267:                                              uint64_t Address,
 268:                                              const MCDisassembler *Decoder) {
 269:   unsigned DstA, SrcB;
 270:   LLVM_DEBUG(dbgs() << "Decoding LdRLimm\n");
 271:   DstA = decodeAField(Insn);
 272:   DecodeGPR32RegisterClass(Inst, DstA, Address, Decoder);
 273:   SrcB = decodeBField(Insn);
 274:   DecodeGPR32RegisterClass(Inst, SrcB, Address, Decoder);
 275:   if (decodeCField(Insn) != 62) {
 276:     LLVM_DEBUG(dbgs() << "Decoding LdRLimm found non-limm register.");
 277:     return MCDisassembler::Fail;
 278:   }
 279:   Inst.addOperand(MCOperand::createImm((uint32_t)(Insn >> 32)));
 280:   return MCDisassembler::Success;
```

- EN: Function bodies or method definitions such as DecodeLdRLImmInstruction contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodeLdRLImmInstruction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 281-300

```cpp
 281: }
 282: 
 283: static DecodeStatus DecodeMoveHRegInstruction(MCInst &Inst, uint64_t Insn,
 284:                                               uint64_t Address,
 285:                                               const MCDisassembler *Decoder) {
 286:   LLVM_DEBUG(dbgs() << "Decoding MOV_S h-register\n");
 287:   using Field = decltype(Insn);
 288:   Field H = fieldFromInstruction(Insn, 5, 3) |
 289:             (fieldFromInstruction(Insn, 0, 2) << 3);
 290:   Field G = fieldFromInstruction(Insn, 8, 3) |
 291:             (fieldFromInstruction(Insn, 3, 2) << 3);
 292: 
 293:   auto DecodeRegisterOrImm = [&Inst, Address, Decoder](Field RegNum,
 294:                                                        Field Value) {
 295:     if (30 == RegNum) {
 296:       Inst.addOperand(MCOperand::createImm(Value));
 297:       return MCDisassembler::Success;
 298:     }
 299: 
 300:     return DecodeGPR32RegisterClass(Inst, RegNum, Address, Decoder);
```

- EN: Function bodies or method definitions such as DecodeMoveHRegInstruction contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodeMoveHRegInstruction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 301-320

```cpp
 301:   };
 302: 
 303:   if (MCDisassembler::Success != DecodeRegisterOrImm(G, 0))
 304:     return MCDisassembler::Fail;
 305: 
 306:   return DecodeRegisterOrImm(H, Insn >> 16u);
 307: }
 308: 
 309: static DecodeStatus DecodeCCRU6Instruction(MCInst &Inst, uint64_t Insn,
 310:                                            uint64_t Address,
 311:                                            const MCDisassembler *Decoder) {
 312:   unsigned DstB;
 313:   LLVM_DEBUG(dbgs() << "Decoding CCRU6 instruction:\n");
 314:   DstB = decodeBField(Insn);
 315:   DecodeGPR32RegisterClass(Inst, DstB, Address, Decoder);
 316:   using Field = decltype(Insn);
 317:   Field U6Field = fieldFromInstruction(Insn, 6, 6);
 318:   Inst.addOperand(MCOperand::createImm(U6Field));
 319:   Field CCField = fieldFromInstruction(Insn, 0, 4);
 320:   Inst.addOperand(MCOperand::createImm(CCField));
```

- EN: Function bodies or method definitions such as DecodeCCRU6Instruction contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodeCCRU6Instruction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 321-340

```cpp
 321:   return MCDisassembler::Success;
 322: }
 323: 
 324: static DecodeStatus DecodeSOPwithRU6(MCInst &Inst, uint64_t Insn,
 325:                                      uint64_t Address,
 326:                                      const MCDisassembler *Decoder) {
 327:   unsigned DstB = decodeBField(Insn);
 328:   DecodeGPR32RegisterClass(Inst, DstB, Address, Decoder);
 329:   using Field = decltype(Insn);
 330:   Field U6 = fieldFromInstruction(Insn, 6, 6);
 331:   Inst.addOperand(MCOperand::createImm(U6));
 332:   return MCDisassembler::Success;
 333: }
 334: 
 335: static DecodeStatus DecodeSOPwithRS12(MCInst &Inst, uint64_t Insn,
 336:                                       uint64_t Address,
 337:                                       const MCDisassembler *Decoder) {
 338:   unsigned DstB = decodeBField(Insn);
 339:   DecodeGPR32RegisterClass(Inst, DstB, Address, Decoder);
 340:   using Field = decltype(Insn);
```

- EN: Function bodies or method definitions such as DecodeSOPwithRU6, DecodeSOPwithRS12 contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodeSOPwithRU6, DecodeSOPwithRS12 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 341-360

```cpp
 341:   Field Lower = fieldFromInstruction(Insn, 6, 6);
 342:   Field Upper = fieldFromInstruction(Insn, 0, 5);
 343:   Field Sign = fieldFromInstruction(Insn, 5, 1) ? -1 : 1;
 344:   Field Result = Sign * ((Upper << 6) + Lower);
 345:   Inst.addOperand(MCOperand::createImm(Result));
 346:   return MCDisassembler::Success;
 347: }
 348: 
 349: DecodeStatus ARCDisassembler::getInstruction(MCInst &Instr, uint64_t &Size,
 350:                                              ArrayRef<uint8_t> Bytes,
 351:                                              uint64_t Address,
 352:                                              raw_ostream &cStream) const {
 353:   MCDisassembler::DecodeStatus Result;
 354:   if (Bytes.size() < 2) {
 355:     Size = 0;
 356:     return Fail;
 357:   }
 358:   uint8_t DecodeByte = (Bytes[1] & 0xF7) >> 3;
 359:   // 0x00 -> 0x07 are 32-bit instructions.
 360:   // 0x08 -> 0x1F are 16-bit instructions.
```

- EN: Function bodies or method definitions such as getInstruction contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: getInstruction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 361-380

```cpp
 361:   if (DecodeByte < 0x08) {
 362:     // 32-bit instruction.
 363:     if (Bytes.size() < 4) {
 364:       // Did we decode garbage?
 365:       Size = 0;
 366:       return Fail;
 367:     }
 368:     if (Bytes.size() >= 8) {
 369:       // Attempt to decode 64-bit instruction.
 370:       uint64_t Insn64;
 371:       if (!readInstruction64(Bytes, Address, Size, Insn64))
 372:         return Fail;
 373:       Result =
 374:           decodeInstruction(DecoderTable64, Instr, Insn64, Address, this, STI);
 375:       if (Success == Result) {
 376:         LLVM_DEBUG(dbgs() << "Successfully decoded 64-bit instruction.");
 377:         return Result;
 378:       }
 379:       LLVM_DEBUG(dbgs() << "Not a 64-bit instruction, falling back to 32-bit.");
 380:     }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 381-400

```cpp
 381:     uint32_t Insn32;
 382:     if (!readInstruction32(Bytes, Address, Size, Insn32)) {
 383:       return Fail;
 384:     }
 385:     // Calling the auto-generated decoder function.
 386:     return decodeInstruction(DecoderTable32, Instr, Insn32, Address, this, STI);
 387:   } else {
 388:     if (Bytes.size() >= 6) {
 389:       // Attempt to treat as instr. with limm data.
 390:       uint64_t Insn48;
 391:       if (!readInstruction48(Bytes, Address, Size, Insn48))
 392:         return Fail;
 393:       Result =
 394:           decodeInstruction(DecoderTable48, Instr, Insn48, Address, this, STI);
 395:       if (Success == Result) {
 396:         LLVM_DEBUG(
 397:             dbgs() << "Successfully decoded 16-bit instruction with limm.");
 398:         return Result;
 399:       }
 400:       LLVM_DEBUG(
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 401-420

```cpp
 401:           dbgs() << "Not a 16-bit instruction with limm, try without it.");
 402:     }
 403: 
 404:     uint32_t Insn16;
 405:     if (!readInstruction16(Bytes, Address, Size, Insn16))
 406:       return Fail;
 407: 
 408:     // Calling the auto-generated decoder function.
 409:     return decodeInstruction(DecoderTable16, Instr, Insn16, Address, this, STI);
 410:   }
 411: }
 412: 
 413: static MCDisassembler *createARCDisassembler(const Target &T,
 414:                                              const MCSubtargetInfo &STI,
 415:                                              MCContext &Ctx) {
 416:   return new ARCDisassembler(STI, Ctx, T.createMCInstrInfo());
 417: }
 418: 
 419: extern "C" LLVM_EXTERNAL_VISIBILITY void LLVMInitializeARCDisassembler() {
 420:   // Register the disassembler.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 421-423

```cpp
 421:   TargetRegistry::RegisterMCDisassembler(getTheARCTarget(),
 422:                                          createARCDisassembler);
 423: }
```

- EN: In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

## Key Concepts / 关键概念

- Decode tables / 解码表
- MCInst reconstruction / MCInst 重建
- MC instruction representation / MC 指令表示
- Register classes / 寄存器类
- CPU feature modelling / CPU 特性建模
- Instruction decoding / 指令解码

## Dependencies / 依赖关系

- Direct includes / 直接包含: `ARC.h`, `ARCRegisterInfo.h`, `MCTargetDesc/ARCMCTargetDesc.h`, `TargetInfo/ARCTargetInfo.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCDecoder.h`, `llvm/MC/MCDecoderOps.h`, `llvm/MC/MCDisassembler/MCDisassembler.h`
- LLVM subsystems / LLVM 子系统: LLVM MC
- Generated or companion files / 生成或配套文件: `ARCGenDisassemblerTables.inc`
