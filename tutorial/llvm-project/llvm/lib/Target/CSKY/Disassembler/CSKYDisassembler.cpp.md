# CSKYDisassembler.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/Disassembler/CSKYDisassembler.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file implements the CSKYDisassembler class.
- 目的（中文）: 实现目标反汇编器，将二进制指令解码回 MC 指令。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- CSKYDisassembler.cpp - Disassembler for CSKY ----------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file implements the CSKYDisassembler class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "MCTargetDesc/CSKYBaseInfo.h"
  14: #include "MCTargetDesc/CSKYMCTargetDesc.h"
  15: #include "TargetInfo/CSKYTargetInfo.h"
  16: #include "llvm/ADT/DenseMap.h"
  17: #include "llvm/MC/MCContext.h"
  18: #include "llvm/MC/MCDecoder.h"
  19: #include "llvm/MC/MCDecoderOps.h"
  20: #include "llvm/MC/MCDisassembler/MCDisassembler.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 21-40

```cpp
  21: #include "llvm/MC/MCInst.h"
  22: #include "llvm/MC/MCInstrInfo.h"
  23: #include "llvm/MC/MCRegisterInfo.h"
  24: #include "llvm/MC/MCSubtargetInfo.h"
  25: #include "llvm/MC/TargetRegistry.h"
  26: #include "llvm/Support/Endian.h"
  27: 
  28: using namespace llvm;
  29: using namespace llvm::MCD;
  30: 
  31: #define DEBUG_TYPE "csky-disassembler"
  32: 
  33: typedef MCDisassembler::DecodeStatus DecodeStatus;
  34: 
  35: namespace {
  36: class CSKYDisassembler : public MCDisassembler {
  37:   std::unique_ptr<MCInstrInfo const> const MCII;
  38:   mutable StringRef symbolName;
  39: 
  40: public:
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as CSKYDisassembler, which organize the target-specific behavior exposed by the file. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 CSKYDisassembler 等接口或数据结构，用于组织该文件暴露的目标专用行为。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 41-60

```cpp
  41:   CSKYDisassembler(const MCSubtargetInfo &STI, MCContext &Ctx,
  42:                    MCInstrInfo const *MCII);
  43: 
  44:   DecodeStatus getInstruction(MCInst &Instr, uint64_t &Size,
  45:                               ArrayRef<uint8_t> Bytes, uint64_t Address,
  46:                               raw_ostream &CStream) const override;
  47: };
  48: } // end anonymous namespace
  49: 
  50: CSKYDisassembler::CSKYDisassembler(const MCSubtargetInfo &STI, MCContext &Ctx,
  51:                                    MCInstrInfo const *MCII)
  52:     : MCDisassembler(STI, Ctx), MCII(MCII) {}
  53: 
  54: static MCDisassembler *createCSKYDisassembler(const Target &T,
  55:                                               const MCSubtargetInfo &STI,
  56:                                               MCContext &Ctx) {
  57:   return new CSKYDisassembler(STI, Ctx, T.createMCInstrInfo());
  58: }
  59: 
  60: extern "C" LLVM_EXTERNAL_VISIBILITY void LLVMInitializeCSKYDisassembler() {
```

- EN: Function bodies or method definitions such as MCDisassembler contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: MCDisassembler 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 61-80

```cpp
  61:   TargetRegistry::RegisterMCDisassembler(getTheCSKYTarget(),
  62:                                          createCSKYDisassembler);
  63: }
  64: 
  65: static const uint16_t GPRDecoderTable[] = {
  66:     CSKY::R0,  CSKY::R1,  CSKY::R2,  CSKY::R3,  CSKY::R4,  CSKY::R5,  CSKY::R6,
  67:     CSKY::R7,  CSKY::R8,  CSKY::R9,  CSKY::R10, CSKY::R11, CSKY::R12, CSKY::R13,
  68:     CSKY::R14, CSKY::R15, CSKY::R16, CSKY::R17, CSKY::R18, CSKY::R19, CSKY::R20,
  69:     CSKY::R21, CSKY::R22, CSKY::R23, CSKY::R24, CSKY::R25, CSKY::R26, CSKY::R27,
  70:     CSKY::R28, CSKY::R29, CSKY::R30, CSKY::R31};
  71: 
  72: static const uint16_t GPRPairDecoderTable[] = {
  73:     CSKY::R0_R1,   CSKY::R1_R2,   CSKY::R2_R3,   CSKY::R3_R4,   CSKY::R4_R5,
  74:     CSKY::R5_R6,   CSKY::R6_R7,   CSKY::R7_R8,   CSKY::R8_R9,   CSKY::R9_R10,
  75:     CSKY::R10_R11, CSKY::R11_R12, CSKY::R12_R13, CSKY::R13_R14, CSKY::R14_R15,
  76:     CSKY::R15_R16, CSKY::R16_R17, CSKY::R17_R18, CSKY::R18_R19, CSKY::R19_R20,
  77:     CSKY::R20_R21, CSKY::R21_R22, CSKY::R22_R23, CSKY::R23_R24, CSKY::R24_R25,
  78:     CSKY::R25_R26, CSKY::R26_R27, CSKY::R27_R28, CSKY::R28_R29, CSKY::R29_R30,
  79:     CSKY::R30_R31, CSKY::R31_R32};
  80: 
```

- EN: In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 81-100

```cpp
  81: static const uint16_t FPR32DecoderTable[] = {
  82:     CSKY::F0_32,  CSKY::F1_32,  CSKY::F2_32,  CSKY::F3_32,  CSKY::F4_32,
  83:     CSKY::F5_32,  CSKY::F6_32,  CSKY::F7_32,  CSKY::F8_32,  CSKY::F9_32,
  84:     CSKY::F10_32, CSKY::F11_32, CSKY::F12_32, CSKY::F13_32, CSKY::F14_32,
  85:     CSKY::F15_32, CSKY::F16_32, CSKY::F17_32, CSKY::F18_32, CSKY::F19_32,
  86:     CSKY::F20_32, CSKY::F21_32, CSKY::F22_32, CSKY::F23_32, CSKY::F24_32,
  87:     CSKY::F25_32, CSKY::F26_32, CSKY::F27_32, CSKY::F28_32, CSKY::F29_32,
  88:     CSKY::F30_32, CSKY::F31_32};
  89: 
  90: static const uint16_t FPR64DecoderTable[] = {
  91:     CSKY::F0_64,  CSKY::F1_64,  CSKY::F2_64,  CSKY::F3_64,  CSKY::F4_64,
  92:     CSKY::F5_64,  CSKY::F6_64,  CSKY::F7_64,  CSKY::F8_64,  CSKY::F9_64,
  93:     CSKY::F10_64, CSKY::F11_64, CSKY::F12_64, CSKY::F13_64, CSKY::F14_64,
  94:     CSKY::F15_64, CSKY::F16_64, CSKY::F17_64, CSKY::F18_64, CSKY::F19_64,
  95:     CSKY::F20_64, CSKY::F21_64, CSKY::F22_64, CSKY::F23_64, CSKY::F24_64,
  96:     CSKY::F25_64, CSKY::F26_64, CSKY::F27_64, CSKY::F28_64, CSKY::F29_64,
  97:     CSKY::F30_64, CSKY::F31_64};
  98: 
  99: static const uint16_t FPR128DecoderTable[] = {
 100:     CSKY::F0_128,  CSKY::F1_128,  CSKY::F2_128,  CSKY::F3_128,  CSKY::F4_128,
```

- EN: In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 101-120

```cpp
 101:     CSKY::F5_128,  CSKY::F6_128,  CSKY::F7_128,  CSKY::F8_128,  CSKY::F9_128,
 102:     CSKY::F10_128, CSKY::F11_128, CSKY::F12_128, CSKY::F13_128, CSKY::F14_128,
 103:     CSKY::F15_128, CSKY::F16_128, CSKY::F17_128, CSKY::F18_128, CSKY::F19_128,
 104:     CSKY::F20_128, CSKY::F21_128, CSKY::F22_128, CSKY::F23_128, CSKY::F24_128,
 105:     CSKY::F25_128, CSKY::F26_128, CSKY::F27_128, CSKY::F28_128, CSKY::F29_128,
 106:     CSKY::F30_128, CSKY::F31_128};
 107: 
 108: static DecodeStatus DecodeGPRRegisterClass(MCInst &Inst, uint64_t RegNo,
 109:                                            uint64_t Address,
 110:                                            const MCDisassembler *Decoder) {
 111:   if (RegNo >= 32)
 112:     return MCDisassembler::Fail;
 113: 
 114:   Inst.addOperand(MCOperand::createReg(GPRDecoderTable[RegNo]));
 115:   return MCDisassembler::Success;
 116: }
 117: 
 118: static DecodeStatus DecodeFPR32RegisterClass(MCInst &Inst, uint64_t RegNo,
 119:                                              uint64_t Address,
 120:                                              const MCDisassembler *Decoder) {
```

- EN: Function bodies or method definitions such as DecodeGPRRegisterClass, DecodeFPR32RegisterClass contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodeGPRRegisterClass, DecodeFPR32RegisterClass 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 121-140

```cpp
 121:   if (RegNo >= 32)
 122:     return MCDisassembler::Fail;
 123: 
 124:   Inst.addOperand(MCOperand::createReg(FPR32DecoderTable[RegNo]));
 125:   return MCDisassembler::Success;
 126: }
 127: 
 128: static DecodeStatus DecodesFPR32RegisterClass(MCInst &Inst, uint64_t RegNo,
 129:                                               uint64_t Address,
 130:                                               const MCDisassembler *Decoder) {
 131:   if (RegNo >= 16)
 132:     return MCDisassembler::Fail;
 133: 
 134:   Inst.addOperand(MCOperand::createReg(FPR32DecoderTable[RegNo]));
 135:   return MCDisassembler::Success;
 136: }
 137: 
 138: static DecodeStatus DecodesFPR64RegisterClass(MCInst &Inst, uint64_t RegNo,
 139:                                               uint64_t Address,
 140:                                               const MCDisassembler *Decoder) {
```

- EN: Function bodies or method definitions such as DecodesFPR32RegisterClass, DecodesFPR64RegisterClass contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodesFPR32RegisterClass, DecodesFPR64RegisterClass 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 141-160

```cpp
 141:   if (RegNo >= 16)
 142:     return MCDisassembler::Fail;
 143: 
 144:   Inst.addOperand(MCOperand::createReg(FPR64DecoderTable[RegNo]));
 145:   return MCDisassembler::Success;
 146: }
 147: 
 148: static DecodeStatus DecodesFPR64_VRegisterClass(MCInst &Inst, uint64_t RegNo,
 149:                                                 uint64_t Address,
 150:                                                 const MCDisassembler *Decoder) {
 151:   if (RegNo >= 16)
 152:     return MCDisassembler::Fail;
 153: 
 154:   Inst.addOperand(MCOperand::createReg(FPR64DecoderTable[RegNo]));
 155:   return MCDisassembler::Success;
 156: }
 157: 
 158: static DecodeStatus DecodeFPR64RegisterClass(MCInst &Inst, uint64_t RegNo,
 159:                                              uint64_t Address,
 160:                                              const MCDisassembler *Decoder) {
```

- EN: Function bodies or method definitions such as DecodesFPR64_VRegisterClass, DecodeFPR64RegisterClass contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodesFPR64_VRegisterClass, DecodeFPR64RegisterClass 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 161-180

```cpp
 161:   if (RegNo >= 32)
 162:     return MCDisassembler::Fail;
 163: 
 164:   Inst.addOperand(MCOperand::createReg(FPR64DecoderTable[RegNo]));
 165:   return MCDisassembler::Success;
 166: }
 167: 
 168: // TODO
 169: [[maybe_unused]]
 170: static DecodeStatus DecodesFPR128RegisterClass(MCInst &Inst, uint64_t RegNo,
 171:                                                uint64_t Address,
 172:                                                const MCDisassembler *Decoder) {
 173:   if (RegNo >= 16)
 174:     return MCDisassembler::Fail;
 175: 
 176:   Inst.addOperand(MCOperand::createReg(FPR128DecoderTable[RegNo]));
 177:   return MCDisassembler::Success;
 178: }
 179: 
 180: static DecodeStatus DecodesGPRRegisterClass(MCInst &Inst, uint64_t RegNo,
```

- EN: Function bodies or method definitions such as DecodesFPR128RegisterClass contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodesFPR128RegisterClass 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 181-200

```cpp
 181:                                             uint64_t Address,
 182:                                             const MCDisassembler *Decoder) {
 183:   if (RegNo >= 16)
 184:     return MCDisassembler::Fail;
 185: 
 186:   Inst.addOperand(MCOperand::createReg(GPRDecoderTable[RegNo]));
 187:   return MCDisassembler::Success;
 188: }
 189: 
 190: static DecodeStatus DecodemGPRRegisterClass(MCInst &Inst, uint64_t RegNo,
 191:                                             uint64_t Address,
 192:                                             const MCDisassembler *Decoder) {
 193:   if (RegNo >= 8)
 194:     return MCDisassembler::Fail;
 195: 
 196:   Inst.addOperand(MCOperand::createReg(GPRDecoderTable[RegNo]));
 197:   return MCDisassembler::Success;
 198: }
 199: 
 200: static DecodeStatus DecodeGPRSPRegisterClass(MCInst &Inst,
```

- EN: Function bodies or method definitions such as DecodemGPRRegisterClass contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodemGPRRegisterClass 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 201-220

```cpp
 201:                                              const MCDisassembler *Decoder) {
 202:   Inst.addOperand(MCOperand::createReg(CSKY::R14));
 203:   return MCDisassembler::Success;
 204: }
 205: 
 206: static DecodeStatus DecodeGPRPairRegisterClass(MCInst &Inst, uint64_t RegNo,
 207:                                                uint64_t Address,
 208:                                                const MCDisassembler *Decoder) {
 209:   const FeatureBitset &FeatureBits =
 210:       Decoder->getSubtargetInfo().getFeatureBits();
 211:   bool hasHighReg = FeatureBits[CSKY::FeatureHighreg];
 212: 
 213:   if (RegNo >= 32 || (!hasHighReg && RegNo >= 16))
 214:     return MCDisassembler::Fail;
 215: 
 216:   Inst.addOperand(MCOperand::createReg(GPRPairDecoderTable[RegNo]));
 217:   return MCDisassembler::Success;
 218: }
 219: 
 220: static DecodeStatus DecodeCARRYRegisterClass(MCInst &Inst,
```

- EN: Function bodies or method definitions such as DecodeGPRPairRegisterClass contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodeGPRPairRegisterClass 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 221-240

```cpp
 221:                                              const MCDisassembler *Decoder) {
 222:   Inst.addOperand(MCOperand::createReg(CSKY::C));
 223:   return MCDisassembler::Success;
 224: }
 225: 
 226: template <unsigned N, unsigned S>
 227: static DecodeStatus decodeUImmOperand(MCInst &Inst, uint64_t Imm,
 228:                                       int64_t Address,
 229:                                       const MCDisassembler *Decoder) {
 230:   assert(isUInt<N>(Imm) && "Invalid immediate");
 231:   Inst.addOperand(MCOperand::createImm(Imm << S));
 232:   return MCDisassembler::Success;
 233: }
 234: 
 235: template <unsigned N>
 236: static DecodeStatus decodeOImmOperand(MCInst &Inst, uint64_t Imm,
 237:                                       int64_t Address,
 238:                                       const MCDisassembler *Decoder) {
 239:   assert(isUInt<N>(Imm) && "Invalid immediate");
 240:   Inst.addOperand(MCOperand::createImm(Imm + 1));
```

- EN: Function bodies or method definitions such as decodeUImmOperand, decodeOImmOperand contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: decodeUImmOperand, decodeOImmOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 241-260

```cpp
 241:   return MCDisassembler::Success;
 242: }
 243: 
 244: static DecodeStatus decodeLRW16Imm8(MCInst &Inst, uint64_t Imm, int64_t Address,
 245:                                     const MCDisassembler *Decoder) {
 246:   assert(isUInt<8>(Imm) && "Invalid immediate");
 247:   if ((Imm >> 7) & 0x1) {
 248:     Inst.addOperand(MCOperand::createImm((Imm & 0x7F) << 2));
 249:   } else {
 250:     uint64_t V = ((Imm ^ 0xFFFFFFFF) & 0xFF);
 251:     Inst.addOperand(MCOperand::createImm(V << 2));
 252:   }
 253: 
 254:   return MCDisassembler::Success;
 255: }
 256: 
 257: static DecodeStatus decodeJMPIXImmOperand(MCInst &Inst, uint64_t Imm,
 258:                                           int64_t Address,
 259:                                           const MCDisassembler *Decoder) {
 260:   assert(isUInt<2>(Imm) && "Invalid immediate");
```

- EN: Function bodies or method definitions such as decodeLRW16Imm8, decodeJMPIXImmOperand contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: decodeLRW16Imm8, decodeJMPIXImmOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 261-280

```cpp
 261: 
 262:   if (Imm == 0)
 263:     Inst.addOperand(MCOperand::createImm(16));
 264:   else if (Imm == 1)
 265:     Inst.addOperand(MCOperand::createImm(24));
 266:   else if (Imm == 2)
 267:     Inst.addOperand(MCOperand::createImm(32));
 268:   else if (Imm == 3)
 269:     Inst.addOperand(MCOperand::createImm(40));
 270:   else
 271:     return MCDisassembler::Fail;
 272: 
 273:   return MCDisassembler::Success;
 274: }
 275: 
 276: static DecodeStatus DecodeRegSeqOperand(MCInst &Inst, uint64_t Imm,
 277:                                         int64_t Address,
 278:                                         const MCDisassembler *Decoder) {
 279:   assert(isUInt<10>(Imm) && "Invalid immediate");
 280: 
```

- EN: Function bodies or method definitions such as DecodeRegSeqOperand contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodeRegSeqOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 281-300

```cpp
 281:   auto Imm5 = Imm & 0x1f;
 282:   auto Ry = (Imm >> 5) & 0x1f;
 283: 
 284:   if (DecodeGPRRegisterClass(Inst, Ry, Address, Decoder) ==
 285:       MCDisassembler::Fail)
 286:     return MCDisassembler::Fail;
 287: 
 288:   Inst.addOperand(MCOperand::createReg(GPRDecoderTable[Ry + Imm5]));
 289: 
 290:   return MCDisassembler::Success;
 291: }
 292: 
 293: static DecodeStatus DecodeRegSeqOperandF1(MCInst &Inst, uint64_t Imm,
 294:                                           int64_t Address,
 295:                                           const MCDisassembler *Decoder) {
 296:   assert(isUInt<10>(Imm) && "Invalid immediate");
 297: 
 298:   auto Imm5 = Imm & 0x1f;
 299:   auto Ry = (Imm >> 5) & 0x1f;
 300: 
```

- EN: Function bodies or method definitions such as DecodeRegSeqOperandF1 contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodeRegSeqOperandF1 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 301-320

```cpp
 301:   if (DecodesFPR32RegisterClass(Inst, Ry, Address, Decoder) ==
 302:       MCDisassembler::Fail)
 303:     return MCDisassembler::Fail;
 304: 
 305:   Inst.addOperand(MCOperand::createReg(FPR32DecoderTable[Ry + Imm5]));
 306: 
 307:   return MCDisassembler::Success;
 308: }
 309: 
 310: static DecodeStatus DecodeRegSeqOperandD1(MCInst &Inst, uint64_t Imm,
 311:                                           int64_t Address,
 312:                                           const MCDisassembler *Decoder) {
 313:   assert(isUInt<10>(Imm) && "Invalid immediate");
 314: 
 315:   auto Imm5 = Imm & 0x1f;
 316:   auto Ry = (Imm >> 5) & 0x1f;
 317: 
 318:   if (DecodesFPR64RegisterClass(Inst, Ry, Address, Decoder) ==
 319:       MCDisassembler::Fail)
 320:     return MCDisassembler::Fail;
```

- EN: Function bodies or method definitions such as DecodeRegSeqOperandD1 contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodeRegSeqOperandD1 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 321-340

```cpp
 321: 
 322:   Inst.addOperand(MCOperand::createReg(FPR64DecoderTable[Ry + Imm5]));
 323: 
 324:   return MCDisassembler::Success;
 325: }
 326: 
 327: static DecodeStatus DecodeRegSeqOperandF2(MCInst &Inst, uint64_t Imm,
 328:                                           int64_t Address,
 329:                                           const MCDisassembler *Decoder) {
 330:   assert(isUInt<10>(Imm) && "Invalid immediate");
 331: 
 332:   auto Imm5 = Imm & 0x1f;
 333:   auto Ry = (Imm >> 5) & 0x1f;
 334: 
 335:   if (DecodeFPR32RegisterClass(Inst, Ry, Address, Decoder) ==
 336:       MCDisassembler::Fail)
 337:     return MCDisassembler::Fail;
 338: 
 339:   Inst.addOperand(MCOperand::createReg(FPR32DecoderTable[Ry + Imm5]));
 340: 
```

- EN: Function bodies or method definitions such as DecodeRegSeqOperandF2 contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodeRegSeqOperandF2 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 341-360

```cpp
 341:   return MCDisassembler::Success;
 342: }
 343: 
 344: static DecodeStatus DecodeRegSeqOperandD2(MCInst &Inst, uint64_t Imm,
 345:                                           int64_t Address,
 346:                                           const MCDisassembler *Decoder) {
 347:   assert(isUInt<10>(Imm) && "Invalid immediate");
 348: 
 349:   auto Imm5 = Imm & 0x1f;
 350:   auto Ry = (Imm >> 5) & 0x1f;
 351: 
 352:   if (DecodeFPR64RegisterClass(Inst, Ry, Address, Decoder) ==
 353:       MCDisassembler::Fail)
 354:     return MCDisassembler::Fail;
 355: 
 356:   Inst.addOperand(MCOperand::createReg(FPR64DecoderTable[Ry + Imm5]));
 357: 
 358:   return MCDisassembler::Success;
 359: }
 360: 
```

- EN: Function bodies or method definitions such as DecodeRegSeqOperandD2 contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodeRegSeqOperandD2 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 361-380

```cpp
 361: static DecodeStatus decodeImmShiftOpValue(MCInst &Inst, uint64_t Imm,
 362:                                           int64_t Address,
 363:                                           const MCDisassembler *Decoder) {
 364:   Inst.addOperand(MCOperand::createImm(Log2_64(Imm)));
 365:   return MCDisassembler::Success;
 366: }
 367: 
 368: template <unsigned N, unsigned S>
 369: static DecodeStatus decodeSImmOperand(MCInst &Inst, uint64_t Imm,
 370:                                       int64_t Address,
 371:                                       const MCDisassembler *Decoder) {
 372:   assert(isUInt<N>(Imm) && "Invalid immediate");
 373:   // Sign-extend the number in the bottom N bits of Imm
 374:   Inst.addOperand(MCOperand::createImm(SignExtend64<N>(Imm) << S));
 375:   return MCDisassembler::Success;
 376: }
 377: 
 378: #include "CSKYGenDisassemblerTables.inc"
 379: 
 380: static bool decodeFPUV3Instruction(MCInst &MI, uint32_t insn, uint64_t Address,
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. Function bodies or method definitions such as decodeImmShiftOpValue, decodeSImmOperand contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 decodeImmShiftOpValue, decodeSImmOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 381-400

```cpp
 381:                                    const MCDisassembler *DisAsm,
 382:                                    const MCSubtargetInfo &STI) {
 383:   LLVM_DEBUG(dbgs() << "Trying CSKY 32-bit fpuv3 table :\n");
 384:   if (!STI.hasFeature(CSKY::FeatureFPUV3_HF) &&
 385:       !STI.hasFeature(CSKY::FeatureFPUV3_SF) &&
 386:       !STI.hasFeature(CSKY::FeatureFPUV3_DF))
 387:     return false;
 388: 
 389:   DecodeStatus Result =
 390:       decodeInstruction(DecoderTableFPUV332, MI, insn, Address, DisAsm, STI);
 391: 
 392:   if (Result == MCDisassembler::Fail) {
 393:     MI.clear();
 394:     return false;
 395:   }
 396: 
 397:   return true;
 398: }
 399: 
 400: DecodeStatus CSKYDisassembler::getInstruction(MCInst &MI, uint64_t &Size,
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 401-420

```cpp
 401:                                               ArrayRef<uint8_t> Bytes,
 402:                                               uint64_t Address,
 403:                                               raw_ostream &CS) const {
 404: 
 405:   uint32_t Insn;
 406:   DecodeStatus Result = MCDisassembler::Fail;
 407: 
 408:   Insn = support::endian::read16le(Bytes.data());
 409: 
 410:   if ((Insn >> 14) == 0x3) {
 411:     if (Bytes.size() < 4) {
 412:       Size = 0;
 413:       return MCDisassembler::Fail;
 414:     }
 415:     Insn = (Insn << 16) | support::endian::read16le(&Bytes[2]);
 416: 
 417:     if (decodeFPUV3Instruction(MI, Insn, Address, this, STI))
 418:       Result = MCDisassembler::Success;
 419:     else {
 420:       LLVM_DEBUG(dbgs() << "Trying CSKY 32-bit table :\n");
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 421-440

```cpp
 421:       Result = decodeInstruction(DecoderTable32, MI, Insn, Address, this, STI);
 422:     }
 423: 
 424:     Size = 4;
 425:   } else {
 426:     if (Bytes.size() < 2) {
 427:       Size = 0;
 428:       return MCDisassembler::Fail;
 429:     }
 430:     LLVM_DEBUG(dbgs() << "Trying CSKY 16-bit table :\n");
 431:     Result = decodeInstruction(DecoderTable16, MI, Insn, Address, this, STI);
 432:     Size = 2;
 433:   }
 434: 
 435:   if (MI.getOpcode() == CSKY::INS32) {
 436:     MI.getOperand(3).setImm(MI.getOperand(3).getImm() +
 437:                             MI.getOperand(4).getImm());
 438:   }
 439: 
 440:   return Result;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 441-441

```cpp
 441: }
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

- Direct includes / 直接包含: `MCTargetDesc/CSKYBaseInfo.h`, `MCTargetDesc/CSKYMCTargetDesc.h`, `TargetInfo/CSKYTargetInfo.h`, `llvm/ADT/DenseMap.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCDecoder.h`, `llvm/MC/MCDecoderOps.h`, `llvm/MC/MCDisassembler/MCDisassembler.h`
- LLVM subsystems / LLVM 子系统: LLVM MC
- Generated or companion files / 生成或配套文件: `CSKYGenDisassemblerTables.inc`
