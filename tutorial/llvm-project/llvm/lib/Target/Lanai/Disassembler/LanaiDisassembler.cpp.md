# LanaiDisassembler.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/Lanai/Disassembler/LanaiDisassembler.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file is part of the Lanai Disassembler.
- 目的（中文）: 实现目标反汇编器，将二进制指令解码回 MC 指令。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===- LanaiDisassembler.cpp - Disassembler for Lanai -----------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file is part of the Lanai Disassembler.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "LanaiDisassembler.h"
  14: 
  15: #include "LanaiAluCode.h"
  16: #include "LanaiCondCode.h"
  17: #include "LanaiInstrInfo.h"
  18: #include "TargetInfo/LanaiTargetInfo.h"
  19: #include "llvm/MC/MCDecoder.h"
  20: #include "llvm/MC/MCDecoderOps.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 21-40

```cpp
  21: #include "llvm/MC/MCInst.h"
  22: #include "llvm/MC/MCSubtargetInfo.h"
  23: #include "llvm/MC/TargetRegistry.h"
  24: #include "llvm/Support/Compiler.h"
  25: #include "llvm/Support/Debug.h"
  26: #include "llvm/Support/MathExtras.h"
  27: 
  28: #define DEBUG_TYPE "lanai-disassembler"
  29: 
  30: using namespace llvm;
  31: using namespace llvm::MCD;
  32: 
  33: typedef MCDisassembler::DecodeStatus DecodeStatus;
  34: 
  35: static MCDisassembler *createLanaiDisassembler(const Target & /*T*/,
  36:                                                const MCSubtargetInfo &STI,
  37:                                                MCContext &Ctx) {
  38:   return new LanaiDisassembler(STI, Ctx);
  39: }
  40: 
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 41-60

```cpp
  41: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
  42: LLVMInitializeLanaiDisassembler() {
  43:   // Register the disassembler
  44:   TargetRegistry::RegisterMCDisassembler(getTheLanaiTarget(),
  45:                                          createLanaiDisassembler);
  46: }
  47: 
  48: LanaiDisassembler::LanaiDisassembler(const MCSubtargetInfo &STI, MCContext &Ctx)
  49:     : MCDisassembler(STI, Ctx) {}
  50: 
  51: // clang-format off
  52: static const unsigned GPRDecoderTable[] = {
  53:   Lanai::R0,  Lanai::R1,  Lanai::PC,  Lanai::R3,  Lanai::SP,  Lanai::FP,
  54:   Lanai::R6,  Lanai::R7,  Lanai::RV,  Lanai::R9,  Lanai::RR1, Lanai::RR2,
  55:   Lanai::R12, Lanai::R13, Lanai::R14, Lanai::RCA, Lanai::R16, Lanai::R17,
  56:   Lanai::R18, Lanai::R19, Lanai::R20, Lanai::R21, Lanai::R22, Lanai::R23,
  57:   Lanai::R24, Lanai::R25, Lanai::R26, Lanai::R27, Lanai::R28, Lanai::R29,
  58:   Lanai::R30, Lanai::R31
  59: };
  60: // clang-format on
```

- EN: Function bodies or method definitions such as MCDisassembler contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: MCDisassembler 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 61-80

```cpp
  61: 
  62: DecodeStatus DecodeGPRRegisterClass(MCInst &Inst, unsigned RegNo,
  63:                                     uint64_t /*Address*/,
  64:                                     const MCDisassembler * /*Decoder*/) {
  65:   if (RegNo > 31)
  66:     return MCDisassembler::Fail;
  67: 
  68:   unsigned Reg = GPRDecoderTable[RegNo];
  69:   Inst.addOperand(MCOperand::createReg(Reg));
  70:   return MCDisassembler::Success;
  71: }
  72: 
  73: static DecodeStatus decodeRiMemoryValue(MCInst &Inst, unsigned Insn,
  74:                                         uint64_t Address,
  75:                                         const MCDisassembler *Decoder) {
  76:   // RI memory values encoded using 23 bits:
  77:   //   5 bit register, 16 bit constant
  78:   unsigned Register = (Insn >> 18) & 0x1f;
  79:   Inst.addOperand(MCOperand::createReg(GPRDecoderTable[Register]));
  80:   unsigned Offset = (Insn & 0xffff);
```

- EN: Function bodies or method definitions such as DecodeGPRRegisterClass, decodeRiMemoryValue contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodeGPRRegisterClass, decodeRiMemoryValue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 81-100

```cpp
  81:   Inst.addOperand(MCOperand::createImm(SignExtend32<16>(Offset)));
  82: 
  83:   return MCDisassembler::Success;
  84: }
  85: 
  86: static DecodeStatus decodeRrMemoryValue(MCInst &Inst, unsigned Insn,
  87:                                         uint64_t Address,
  88:                                         const MCDisassembler *Decoder) {
  89:   // RR memory values encoded using 20 bits:
  90:   //   5 bit register, 5 bit register, 2 bit PQ, 3 bit ALU operator, 5 bit JJJJJ
  91:   unsigned Register = (Insn >> 15) & 0x1f;
  92:   Inst.addOperand(MCOperand::createReg(GPRDecoderTable[Register]));
  93:   Register = (Insn >> 10) & 0x1f;
  94:   Inst.addOperand(MCOperand::createReg(GPRDecoderTable[Register]));
  95: 
  96:   return MCDisassembler::Success;
  97: }
  98: 
  99: static DecodeStatus decodeSplsValue(MCInst &Inst, unsigned Insn,
 100:                                     uint64_t Address,
```

- EN: Function bodies or method definitions such as decodeRrMemoryValue contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: decodeRrMemoryValue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 101-120

```cpp
 101:                                     const MCDisassembler *Decoder) {
 102:   // RI memory values encoded using 17 bits:
 103:   //   5 bit register, 10 bit constant
 104:   unsigned Register = (Insn >> 12) & 0x1f;
 105:   Inst.addOperand(MCOperand::createReg(GPRDecoderTable[Register]));
 106:   unsigned Offset = (Insn & 0x3ff);
 107:   Inst.addOperand(MCOperand::createImm(SignExtend32<10>(Offset)));
 108: 
 109:   return MCDisassembler::Success;
 110: }
 111: 
 112: static bool tryAddingSymbolicOperand(int64_t Value, bool IsBranch,
 113:                                      uint64_t Address, uint64_t Offset,
 114:                                      uint64_t Width, MCInst &MI,
 115:                                      const MCDisassembler *Decoder) {
 116:   return Decoder->tryAddingSymbolicOperand(MI, Value, Address, IsBranch, Offset,
 117:                                            Width, /*InstSize=*/0);
 118: }
 119: 
 120: static DecodeStatus decodeBranch(MCInst &MI, unsigned Insn, uint64_t Address,
```

- EN: Function bodies or method definitions such as tryAddingSymbolicOperand contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: tryAddingSymbolicOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 121-140

```cpp
 121:                                  const MCDisassembler *Decoder) {
 122:   if (!tryAddingSymbolicOperand(Insn + Address, false, Address, 2, 23, MI,
 123:                                 Decoder))
 124:     MI.addOperand(MCOperand::createImm(Insn));
 125:   return MCDisassembler::Success;
 126: }
 127: 
 128: static DecodeStatus decodeShiftImm(MCInst &Inst, unsigned Insn,
 129:                                    uint64_t Address,
 130:                                    const MCDisassembler *Decoder) {
 131:   unsigned Offset = (Insn & 0xffff);
 132:   Inst.addOperand(MCOperand::createImm(SignExtend32<16>(Offset)));
 133: 
 134:   return MCDisassembler::Success;
 135: }
 136: 
 137: static DecodeStatus decodePredicateOperand(MCInst &Inst, unsigned Val,
 138:                                            uint64_t Address,
 139:                                            const MCDisassembler *Decoder) {
 140:   if (Val >= LPCC::UNKNOWN)
```

- EN: Function bodies or method definitions such as decodeShiftImm, decodePredicateOperand contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: decodeShiftImm, decodePredicateOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 141-160

```cpp
 141:     return MCDisassembler::Fail;
 142:   Inst.addOperand(MCOperand::createImm(Val));
 143:   return MCDisassembler::Success;
 144: }
 145: 
 146: #include "LanaiGenDisassemblerTables.inc"
 147: 
 148: static DecodeStatus readInstruction32(ArrayRef<uint8_t> Bytes, uint64_t &Size,
 149:                                       uint32_t &Insn) {
 150:   // We want to read exactly 4 bytes of data.
 151:   if (Bytes.size() < 4) {
 152:     Size = 0;
 153:     return MCDisassembler::Fail;
 154:   }
 155: 
 156:   // Encoded as big-endian 32-bit word in the stream.
 157:   Insn =
 158:       (Bytes[0] << 24) | (Bytes[1] << 16) | (Bytes[2] << 8) | (Bytes[3] << 0);
 159: 
 160:   return MCDisassembler::Success;
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. Function bodies or method definitions such as readInstruction32 contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 readInstruction32 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 161-180

```cpp
 161: }
 162: 
 163: static void PostOperandDecodeAdjust(MCInst &Instr, uint32_t Insn) {
 164:   unsigned AluOp = LPAC::ADD;
 165:   // Fix up for pre and post operations.
 166:   int PqShift = -1;
 167:   if (isRMOpcode(Instr.getOpcode()))
 168:     PqShift = 16;
 169:   else if (isSPLSOpcode(Instr.getOpcode()))
 170:     PqShift = 10;
 171:   else if (isRRMOpcode(Instr.getOpcode())) {
 172:     PqShift = 16;
 173:     // Determine RRM ALU op.
 174:     AluOp = (Insn >> 8) & 0x7;
 175:     if (AluOp == 7)
 176:       // Handle JJJJJ
 177:       // 0b10000 or 0b11000
 178:       AluOp |= 0x20 | (((Insn >> 3) & 0xf) << 1);
 179:   }
 180: 
```

- EN: Function bodies or method definitions such as PostOperandDecodeAdjust, if contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: PostOperandDecodeAdjust, if 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 181-200

```cpp
 181:   if (PqShift != -1) {
 182:     unsigned PQ = (Insn >> PqShift) & 0x3;
 183:     switch (PQ) {
 184:     case 0x0:
 185:       if (Instr.getOperand(2).isReg()) {
 186:         Instr.getOperand(2).setReg(Lanai::R0);
 187:       }
 188:       if (Instr.getOperand(2).isImm())
 189:         Instr.getOperand(2).setImm(0);
 190:       break;
 191:     case 0x1:
 192:       AluOp = LPAC::makePostOp(AluOp);
 193:       break;
 194:     case 0x2:
 195:       break;
 196:     case 0x3:
 197:       AluOp = LPAC::makePreOp(AluOp);
 198:       break;
 199:     }
 200:     Instr.addOperand(MCOperand::createImm(AluOp));
```

- EN: Function bodies or method definitions such as if contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: if 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 201-220

```cpp
 201:   }
 202: }
 203: 
 204: DecodeStatus
 205: LanaiDisassembler::getInstruction(MCInst &Instr, uint64_t &Size,
 206:                                   ArrayRef<uint8_t> Bytes, uint64_t Address,
 207:                                   raw_ostream & /*CStream*/) const {
 208:   uint32_t Insn;
 209: 
 210:   DecodeStatus Result = readInstruction32(Bytes, Size, Insn);
 211: 
 212:   if (Result == MCDisassembler::Fail)
 213:     return MCDisassembler::Fail;
 214: 
 215:   // Call auto-generated decoder function
 216:   Result =
 217:       decodeInstruction(DecoderTableLanai32, Instr, Insn, Address, this, STI);
 218: 
 219:   if (Result != MCDisassembler::Fail) {
 220:     PostOperandDecodeAdjust(Instr, Insn);
```

- EN: Function bodies or method definitions such as getInstruction contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: getInstruction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 221-226

```cpp
 221:     Size = 4;
 222:     return Result;
 223:   }
 224: 
 225:   return MCDisassembler::Fail;
 226: }
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

- Direct includes / 直接包含: `LanaiDisassembler.h`, `LanaiAluCode.h`, `LanaiCondCode.h`, `LanaiInstrInfo.h`, `TargetInfo/LanaiTargetInfo.h`, `llvm/MC/MCDecoder.h`, `llvm/MC/MCDecoderOps.h`, `llvm/MC/MCInst.h`
- LLVM subsystems / LLVM 子系统: LLVM MC
- Generated or companion files / 生成或配套文件: `LanaiGenDisassemblerTables.inc`
- Local companions / 本地配套文件: `LanaiDisassembler.h`
