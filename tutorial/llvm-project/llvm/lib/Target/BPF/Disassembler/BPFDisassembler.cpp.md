# BPFDisassembler.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/BPF/Disassembler/BPFDisassembler.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file is part of the BPF Disassembler.
- 目的（中文）: 实现目标反汇编器，将二进制指令解码回 MC 指令。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===- BPFDisassembler.cpp - Disassembler for BPF ---------------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file is part of the BPF Disassembler.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "MCTargetDesc/BPFMCTargetDesc.h"
  14: #include "TargetInfo/BPFTargetInfo.h"
  15: #include "llvm/ADT/ArrayRef.h"
  16: #include "llvm/MC/MCAsmInfo.h"
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
  22: #include "llvm/MC/TargetRegistry.h"
  23: #include "llvm/Support/Compiler.h"
  24: #include "llvm/Support/MathExtras.h"
  25: #include "llvm/TargetParser/SubtargetFeature.h"
  26: #include <cstdint>
  27: 
  28: using namespace llvm;
  29: using namespace llvm::MCD;
  30: 
  31: #define DEBUG_TYPE "bpf-disassembler"
  32: 
  33: typedef MCDisassembler::DecodeStatus DecodeStatus;
  34: 
  35: namespace {
  36: 
  37: /// A disassembler class for BPF.
  38: class BPFDisassembler : public MCDisassembler {
  39: public:
  40:   enum BPF_CLASS {
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as for, BPFDisassembler, which organize the target-specific behavior exposed by the file. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 for, BPFDisassembler 等接口或数据结构，用于组织该文件暴露的目标专用行为。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 41-60

```cpp
  41:     BPF_LD = 0x0,
  42:     BPF_LDX = 0x1,
  43:     BPF_ST = 0x2,
  44:     BPF_STX = 0x3,
  45:     BPF_ALU = 0x4,
  46:     BPF_JMP = 0x5,
  47:     BPF_JMP32 = 0x6,
  48:     BPF_ALU64 = 0x7
  49:   };
  50: 
  51:   enum BPF_SIZE {
  52:     BPF_W = 0x0,
  53:     BPF_H = 0x1,
  54:     BPF_B = 0x2,
  55:     BPF_DW = 0x3
  56:   };
  57: 
  58:   enum BPF_MODE {
  59:     BPF_IMM = 0x0,
  60:     BPF_ABS = 0x1,
```

- EN: In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 61-80

```cpp
  61:     BPF_IND = 0x2,
  62:     BPF_MEM = 0x3,
  63:     BPF_MEMSX = 0x4,
  64:     BPF_ATOMIC = 0x6
  65:   };
  66: 
  67:   BPFDisassembler(const MCSubtargetInfo &STI, MCContext &Ctx)
  68:       : MCDisassembler(STI, Ctx) {}
  69:   ~BPFDisassembler() override = default;
  70: 
  71:   DecodeStatus getInstruction(MCInst &Instr, uint64_t &Size,
  72:                               ArrayRef<uint8_t> Bytes, uint64_t Address,
  73:                               raw_ostream &CStream) const override;
  74: 
  75:   uint8_t getInstClass(uint64_t Inst) const { return (Inst >> 56) & 0x7; };
  76:   uint8_t getInstSize(uint64_t Inst) const { return (Inst >> 59) & 0x3; };
  77:   uint8_t getInstMode(uint64_t Inst) const { return (Inst >> 61) & 0x7; };
  78: };
  79: 
  80: } // end anonymous namespace
```

- EN: Function bodies or method definitions such as MCDisassembler, getInstClass, getInstSize contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: MCDisassembler, getInstClass, getInstSize 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 81-100

```cpp
  81: 
  82: static MCDisassembler *createBPFDisassembler(const Target &T,
  83:                                              const MCSubtargetInfo &STI,
  84:                                              MCContext &Ctx) {
  85:   return new BPFDisassembler(STI, Ctx);
  86: }
  87: 
  88: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
  89: LLVMInitializeBPFDisassembler() {
  90:   // Register the disassembler.
  91:   TargetRegistry::RegisterMCDisassembler(getTheBPFTarget(),
  92:                                          createBPFDisassembler);
  93:   TargetRegistry::RegisterMCDisassembler(getTheBPFleTarget(),
  94:                                          createBPFDisassembler);
  95:   TargetRegistry::RegisterMCDisassembler(getTheBPFbeTarget(),
  96:                                          createBPFDisassembler);
  97: }
  98: 
  99: static const unsigned GPRDecoderTable[] = {
 100:     BPF::R0, BPF::R1, BPF::R2, BPF::R3, BPF::R4,  BPF::R5,
```

- EN: In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 101-120

```cpp
 101:     BPF::R6, BPF::R7, BPF::R8, BPF::R9, BPF::R10, BPF::R11};
 102: 
 103: static DecodeStatus DecodeGPRRegisterClass(MCInst &Inst, unsigned RegNo,
 104:                                            uint64_t /*Address*/,
 105:                                            const MCDisassembler * /*Decoder*/) {
 106:   if (RegNo > 11)
 107:     return MCDisassembler::Fail;
 108: 
 109:   unsigned Reg = GPRDecoderTable[RegNo];
 110:   Inst.addOperand(MCOperand::createReg(Reg));
 111:   return MCDisassembler::Success;
 112: }
 113: 
 114: static const unsigned GPR32DecoderTable[] = {
 115:     BPF::W0, BPF::W1, BPF::W2, BPF::W3, BPF::W4,  BPF::W5,
 116:     BPF::W6, BPF::W7, BPF::W8, BPF::W9, BPF::W10, BPF::W11};
 117: 
 118: static DecodeStatus
 119: DecodeGPR32RegisterClass(MCInst &Inst, unsigned RegNo, uint64_t /*Address*/,
 120:                          const MCDisassembler * /*Decoder*/) {
```

- EN: Function bodies or method definitions such as DecodeGPRRegisterClass, DecodeGPR32RegisterClass contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: DecodeGPRRegisterClass, DecodeGPR32RegisterClass 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 121-140

```cpp
 121:   if (RegNo > 11)
 122:     return MCDisassembler::Fail;
 123: 
 124:   unsigned Reg = GPR32DecoderTable[RegNo];
 125:   Inst.addOperand(MCOperand::createReg(Reg));
 126:   return MCDisassembler::Success;
 127: }
 128: 
 129: static DecodeStatus decodeMemoryOpValue(MCInst &Inst, unsigned Insn,
 130:                                         uint64_t Address,
 131:                                         const MCDisassembler *Decoder) {
 132:   unsigned Register = (Insn >> 16) & 0xf;
 133:   if (Register > 11)
 134:     return MCDisassembler::Fail;
 135: 
 136:   Inst.addOperand(MCOperand::createReg(GPRDecoderTable[Register]));
 137:   unsigned Offset = (Insn & 0xffff);
 138:   Inst.addOperand(MCOperand::createImm(SignExtend32<16>(Offset)));
 139: 
 140:   return MCDisassembler::Success;
```

- EN: Function bodies or method definitions such as decodeMemoryOpValue contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: decodeMemoryOpValue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 141-160

```cpp
 141: }
 142: 
 143: #include "BPFGenDisassemblerTables.inc"
 144: static DecodeStatus readInstruction64(ArrayRef<uint8_t> Bytes, uint64_t Address,
 145:                                       uint64_t &Size, uint64_t &Insn,
 146:                                       bool IsLittleEndian) {
 147:   uint64_t Lo, Hi;
 148: 
 149:   if (Bytes.size() < 8) {
 150:     Size = 0;
 151:     return MCDisassembler::Fail;
 152:   }
 153: 
 154:   Size = 8;
 155:   if (IsLittleEndian) {
 156:     Hi = (Bytes[0] << 24) | (Bytes[1] << 16) | (Bytes[2] << 0) | (Bytes[3] << 8);
 157:     Lo = (Bytes[4] << 0) | (Bytes[5] << 8) | (Bytes[6] << 16) | (Bytes[7] << 24);
 158:   } else {
 159:     Hi = (Bytes[0] << 24) | ((Bytes[1] & 0x0F) << 20) | ((Bytes[1] & 0xF0) << 12) |
 160:          (Bytes[2] << 8) | (Bytes[3] << 0);
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. Function bodies or method definitions such as readInstruction64 contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 readInstruction64 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 161-180

```cpp
 161:     Lo = (Bytes[4] << 24) | (Bytes[5] << 16) | (Bytes[6] << 8) | (Bytes[7] << 0);
 162:   }
 163:   Insn = Make_64(Hi, Lo);
 164: 
 165:   return MCDisassembler::Success;
 166: }
 167: 
 168: DecodeStatus BPFDisassembler::getInstruction(MCInst &Instr, uint64_t &Size,
 169:                                              ArrayRef<uint8_t> Bytes,
 170:                                              uint64_t Address,
 171:                                              raw_ostream &CStream) const {
 172:   bool IsLittleEndian = getContext().getAsmInfo().isLittleEndian();
 173:   uint64_t Insn, Hi;
 174:   DecodeStatus Result;
 175: 
 176:   Result = readInstruction64(Bytes, Address, Size, Insn, IsLittleEndian);
 177:   if (Result == MCDisassembler::Fail) return MCDisassembler::Fail;
 178: 
 179:   uint8_t InstClass = getInstClass(Insn);
 180:   uint8_t InstMode = getInstMode(Insn);
```

- EN: Function bodies or method definitions such as getInstruction contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: getInstruction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 181-200

```cpp
 181:   if ((InstClass == BPF_LDX || InstClass == BPF_STX) &&
 182:       getInstSize(Insn) != BPF_DW &&
 183:       (InstMode == BPF_MEM || InstMode == BPF_ATOMIC) &&
 184:       STI.hasFeature(BPF::ALU32))
 185:     Result = decodeInstruction(DecoderTableBPFALU3264, Instr, Insn, Address,
 186:                                this, STI);
 187:   else
 188:     Result = decodeInstruction(DecoderTableBPF64, Instr, Insn, Address, this,
 189:                                STI);
 190: 
 191:   if (Result == MCDisassembler::Fail) return MCDisassembler::Fail;
 192: 
 193:   switch (Instr.getOpcode()) {
 194:   case BPF::LD_imm64:
 195:   case BPF::LD_pseudo: {
 196:     if (Bytes.size() < 16) {
 197:       Size = 0;
 198:       return MCDisassembler::Fail;
 199:     }
 200:     Size = 16;
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

### Lines 201-215

```cpp
 201:     if (IsLittleEndian)
 202:       Hi = (Bytes[12] << 0) | (Bytes[13] << 8) | (Bytes[14] << 16) | (Bytes[15] << 24);
 203:     else
 204:       Hi = (Bytes[12] << 24) | (Bytes[13] << 16) | (Bytes[14] << 8) | (Bytes[15] << 0);
 205:     auto& Op = Instr.getOperand(1);
 206:     Op.setImm(Make_64(Hi, Op.getImm()));
 207:     break;
 208:   }
 209:   }
 210: 
 211:   return Result;
 212: }
 213: 
 214: typedef DecodeStatus (*DecodeFunc)(MCInst &MI, unsigned insn, uint64_t Address,
 215:                                    const MCDisassembler *Decoder);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it helps decode encoded instructions back into symbolic MC forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它帮助把编码后的指令解码回符号化的 MC 形式。

## Key Concepts / 关键概念

- Decode tables / 解码表
- MCInst reconstruction / MCInst 重建
- MC instruction representation / MC 指令表示
- Register classes / 寄存器类
- CPU feature modelling / CPU 特性建模
- Instruction decoding / 指令解码

## Dependencies / 依赖关系

- Direct includes / 直接包含: `MCTargetDesc/BPFMCTargetDesc.h`, `TargetInfo/BPFTargetInfo.h`, `llvm/ADT/ArrayRef.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCDecoder.h`, `llvm/MC/MCDecoderOps.h`, `llvm/MC/MCDisassembler/MCDisassembler.h`
- LLVM subsystems / LLVM 子系统: LLVM MC
- Generated or companion files / 生成或配套文件: `BPFGenDisassemblerTables.inc`
