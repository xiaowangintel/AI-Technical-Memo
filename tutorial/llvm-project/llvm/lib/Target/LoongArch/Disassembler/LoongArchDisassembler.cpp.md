# LoongArchDisassembler.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/Disassembler/LoongArchDisassembler.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file decodes target machine code into MC instructions for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责将目标机器码解码为 MC 指令。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- LoongArchDisassembler.cpp - Disassembler for LoongArch ------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file implements the LoongArchDisassembler class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "MCTargetDesc/LoongArchMCTargetDesc.h"
  14: #include "TargetInfo/LoongArchTargetInfo.h"
  15: #include "llvm/MC/MCContext.h"
  16: #include "llvm/MC/MCDecoder.h"
  17: #include "llvm/MC/MCDecoderOps.h"
  18: #include "llvm/MC/MCDisassembler/MCDisassembler.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `LoongArchMCTargetDesc.h`, `LoongArchTargetInfo.h`, `MCContext.h`, `MCDecoder.h`, `MCDecoderOps.h`, `MCDisassembler.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `LoongArchMCTargetDesc.h`, `LoongArchTargetInfo.h`, `MCContext.h`, `MCDecoder.h`, `MCDecoderOps.h`, `MCDisassembler.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "llvm/MC/MCInst.h"
  20: #include "llvm/MC/MCInstrInfo.h"
  21: #include "llvm/MC/MCSubtargetInfo.h"
  22: #include "llvm/MC/TargetRegistry.h"
  23: #include "llvm/Support/Compiler.h"
  24: #include "llvm/Support/Endian.h"
  25: 
  26: using namespace llvm;
  27: using namespace llvm::MCD;
  28: 
  29: #define DEBUG_TYPE "loongarch-disassembler"
  30: 
  31: typedef MCDisassembler::DecodeStatus DecodeStatus;
  32: 
  33: namespace {
  34: class LoongArchDisassembler : public MCDisassembler {
  35: public:
  36:   LoongArchDisassembler(const MCSubtargetInfo &STI, MCContext &Ctx)
```
- **EN**: It imports dependencies such as `MCInst.h`, `MCInstrInfo.h`, `MCSubtargetInfo.h`, `TargetRegistry.h`, `Compiler.h`, `Endian.h` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `LoongArchDisassembler`. The range implements or declares functions including `LoongArchDisassembler`.
- **CN**: 它引入了 `MCInst.h`, `MCInstrInfo.h`, `MCSubtargetInfo.h`, `TargetRegistry.h`, `Compiler.h`, `Endian.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `LoongArchDisassembler` 等 TableGen 记录。 这一段实现或声明了 `LoongArchDisassembler` 等函数。

### Lines 37-54 / 第 37-54 行
```cpp
  37:       : MCDisassembler(STI, Ctx) {}
  38: 
  39:   DecodeStatus getInstruction(MCInst &Instr, uint64_t &Size,
  40:                               ArrayRef<uint8_t> Bytes, uint64_t Address,
  41:                               raw_ostream &CStream) const override;
  42: };
  43: } // end namespace
  44: 
  45: static MCDisassembler *createLoongArchDisassembler(const Target &T,
  46:                                                    const MCSubtargetInfo &STI,
  47:                                                    MCContext &Ctx) {
  48:   return new LoongArchDisassembler(STI, Ctx);
  49: }
  50: 
  51: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
  52: LLVMInitializeLoongArchDisassembler() {
  53:   // Register the disassembler for each target.
  54:   TargetRegistry::RegisterMCDisassembler(getTheLoongArch32Target(),
```
- **EN**: The range implements or declares functions including `MCDisassembler`.
- **CN**: 这一段实现或声明了 `MCDisassembler` 等函数。

### Lines 55-72 / 第 55-72 行
```cpp
  55:                                          createLoongArchDisassembler);
  56:   TargetRegistry::RegisterMCDisassembler(getTheLoongArch64Target(),
  57:                                          createLoongArchDisassembler);
  58: }
  59: 
  60: static DecodeStatus DecodeGPRRegisterClass(MCInst &Inst, uint64_t RegNo,
  61:                                            uint64_t Address,
  62:                                            const MCDisassembler *Decoder) {
  63:   if (RegNo >= 32)
  64:     return MCDisassembler::Fail;
  65:   Inst.addOperand(MCOperand::createReg(LoongArch::R0 + RegNo));
  66:   return MCDisassembler::Success;
  67: }
  68: 
  69: static DecodeStatus
  70: DecodeGPRNoR0R1RegisterClass(MCInst &Inst, uint64_t RegNo, uint64_t Address,
  71:                              const MCDisassembler *Decoder) {
  72:   if (RegNo <= 1)
```
- **EN**: The range implements or declares functions including `DecodeGPRRegisterClass`, `DecodeGPRNoR0R1RegisterClass`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `DecodeGPRRegisterClass`, `DecodeGPRNoR0R1RegisterClass` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 73-90 / 第 73-90 行
```cpp
  73:     return MCDisassembler::Fail;
  74:   return DecodeGPRRegisterClass(Inst, RegNo, Address, Decoder);
  75: }
  76: 
  77: static DecodeStatus DecodeFPR32RegisterClass(MCInst &Inst, uint64_t RegNo,
  78:                                              uint64_t Address,
  79:                                              const MCDisassembler *Decoder) {
  80:   if (RegNo >= 32)
  81:     return MCDisassembler::Fail;
  82:   Inst.addOperand(MCOperand::createReg(LoongArch::F0 + RegNo));
  83:   return MCDisassembler::Success;
  84: }
  85: 
  86: static DecodeStatus DecodeFPR64RegisterClass(MCInst &Inst, uint64_t RegNo,
  87:                                              uint64_t Address,
  88:                                              const MCDisassembler *Decoder) {
  89:   if (RegNo >= 32)
  90:     return MCDisassembler::Fail;
```
- **EN**: The range implements or declares functions including `DecodeFPR32RegisterClass`, `DecodeFPR64RegisterClass`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `DecodeFPR32RegisterClass`, `DecodeFPR64RegisterClass` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 91-108 / 第 91-108 行
```cpp
  91:   Inst.addOperand(MCOperand::createReg(LoongArch::F0_64 + RegNo));
  92:   return MCDisassembler::Success;
  93: }
  94: 
  95: static DecodeStatus DecodeCFRRegisterClass(MCInst &Inst, uint64_t RegNo,
  96:                                            uint64_t Address,
  97:                                            const MCDisassembler *Decoder) {
  98:   if (RegNo >= 8)
  99:     return MCDisassembler::Fail;
 100:   Inst.addOperand(MCOperand::createReg(LoongArch::FCC0 + RegNo));
 101:   return MCDisassembler::Success;
 102: }
 103: 
 104: static DecodeStatus DecodeFCSRRegisterClass(MCInst &Inst, uint64_t RegNo,
 105:                                             uint64_t Address,
 106:                                             const MCDisassembler *Decoder) {
 107:   if (RegNo >= 4)
 108:     return MCDisassembler::Fail;
```
- **EN**: The range implements or declares functions including `DecodeCFRRegisterClass`, `DecodeFCSRRegisterClass`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `DecodeCFRRegisterClass`, `DecodeFCSRRegisterClass` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 109-126 / 第 109-126 行
```cpp
 109:   Inst.addOperand(MCOperand::createReg(LoongArch::FCSR0 + RegNo));
 110:   return MCDisassembler::Success;
 111: }
 112: 
 113: static DecodeStatus DecodeLSX128RegisterClass(MCInst &Inst, uint64_t RegNo,
 114:                                               uint64_t Address,
 115:                                               const MCDisassembler *Decoder) {
 116:   if (RegNo >= 32)
 117:     return MCDisassembler::Fail;
 118:   Inst.addOperand(MCOperand::createReg(LoongArch::VR0 + RegNo));
 119:   return MCDisassembler::Success;
 120: }
 121: 
 122: static DecodeStatus DecodeLASX256RegisterClass(MCInst &Inst, uint64_t RegNo,
 123:                                                uint64_t Address,
 124:                                                const MCDisassembler *Decoder) {
 125:   if (RegNo >= 32)
 126:     return MCDisassembler::Fail;
```
- **EN**: The range implements or declares functions including `DecodeLSX128RegisterClass`, `DecodeLASX256RegisterClass`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `DecodeLSX128RegisterClass`, `DecodeLASX256RegisterClass` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 127-144 / 第 127-144 行
```cpp
 127:   Inst.addOperand(MCOperand::createReg(LoongArch::XR0 + RegNo));
 128:   return MCDisassembler::Success;
 129: }
 130: 
 131: static DecodeStatus DecodeSCRRegisterClass(MCInst &Inst, uint64_t RegNo,
 132:                                            uint64_t Address,
 133:                                            const MCDisassembler *Decoder) {
 134:   if (RegNo >= 4)
 135:     return MCDisassembler::Fail;
 136:   Inst.addOperand(MCOperand::createReg(LoongArch::SCR0 + RegNo));
 137:   return MCDisassembler::Success;
 138: }
 139: 
 140: template <unsigned N, int P = 0>
 141: static DecodeStatus decodeUImmOperand(MCInst &Inst, uint64_t Imm,
 142:                                       int64_t Address,
 143:                                       const MCDisassembler *Decoder) {
 144:   assert(isUInt<N>(Imm) && "Invalid immediate");
```
- **EN**: The range implements or declares functions including `DecodeSCRRegisterClass`, `decodeUImmOperand`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `DecodeSCRRegisterClass`, `decodeUImmOperand` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 145-162 / 第 145-162 行
```cpp
 145:   Inst.addOperand(MCOperand::createImm(Imm + P));
 146:   return MCDisassembler::Success;
 147: }
 148: 
 149: template <unsigned N, unsigned S = 0>
 150: static DecodeStatus decodeSImmOperand(MCInst &Inst, uint64_t Imm,
 151:                                       int64_t Address,
 152:                                       const MCDisassembler *Decoder) {
 153:   assert(isUInt<N>(Imm) && "Invalid immediate");
 154:   // Shift left Imm <S> bits, then sign-extend the number in the bottom <N+S>
 155:   // bits.
 156:   Inst.addOperand(MCOperand::createImm(SignExtend64<N + S>(Imm << S)));
 157:   return MCDisassembler::Success;
 158: }
 159: 
 160: // Decode AMSWAP.W and UD, which share the same base encoding.
 161: // If rk == 1 and rd == rj, interpret the instruction as UD;
 162: // otherwise decode as AMSWAP.W.
```
- **EN**: The range implements or declares functions including `decodeSImmOperand`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `decodeSImmOperand` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 163-180 / 第 163-180 行
```cpp
 163: static DecodeStatus DecodeAMOrUDInstruction(MCInst &Inst, unsigned Insn,
 164:                                             uint64_t Address,
 165:                                             const MCDisassembler *Decoder) {
 166:   unsigned Rd = fieldFromInstruction(Insn, 0, 5);
 167:   unsigned Rj = fieldFromInstruction(Insn, 5, 5);
 168:   unsigned Rk = fieldFromInstruction(Insn, 10, 5);
 169: 
 170:   if (Rk == 1 && Rd == Rj) {
 171:     Inst.setOpcode(LoongArch::UD);
 172:     Inst.addOperand(MCOperand::createImm(Rd));
 173:   } else {
 174:     Inst.setOpcode(LoongArch::AMSWAP_W);
 175:     Inst.addOperand(MCOperand::createReg(LoongArch::R0 + Rd));
 176:     Inst.addOperand(MCOperand::createReg(LoongArch::R0 + Rk));
 177:     Inst.addOperand(MCOperand::createReg(LoongArch::R0 + Rj));
 178:   }
 179: 
 180:   return MCDisassembler::Success;
```
- **EN**: The range implements or declares functions including `DecodeAMOrUDInstruction`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `DecodeAMOrUDInstruction` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 181-198 / 第 181-198 行
```cpp
 181: }
 182: 
 183: #include "LoongArchGenDisassemblerTables.inc"
 184: 
 185: DecodeStatus LoongArchDisassembler::getInstruction(MCInst &MI, uint64_t &Size,
 186:                                                    ArrayRef<uint8_t> Bytes,
 187:                                                    uint64_t Address,
 188:                                                    raw_ostream &CS) const {
 189:   uint32_t Insn;
 190:   DecodeStatus Result;
 191: 
 192:   // We want to read exactly 4 bytes of data because all LoongArch instructions
 193:   // are fixed 32 bits.
 194:   if (Bytes.size() < 4) {
 195:     Size = 0;
 196:     return MCDisassembler::Fail;
 197:   }
 198: 
```
- **EN**: It imports dependencies such as `LoongArchGenDisassemblerTables.inc` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `LoongArchDisassembler::getInstruction`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `LoongArchGenDisassemblerTables.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `LoongArchDisassembler::getInstruction` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 199-205 / 第 199-205 行
```cpp
 199:   Insn = support::endian::read32le(Bytes.data());
 200:   // Calling the auto-generated decoder function.
 201:   Result = decodeInstruction(DecoderTable32, MI, Insn, Address, this, STI);
 202:   Size = 4;
 203: 
 204:   return Result;
 205: }
```
- **EN**: This span continues the file's main responsibility: this file decodes target machine code into MC instructions for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **Disassembly**: Decodes binary encodings back into symbolic instructions. / 将二进制编码解码回符号化指令。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `MCTargetDesc/LoongArchMCTargetDesc.h`
- `TargetInfo/LoongArchTargetInfo.h`
- `llvm/MC/MCContext.h`
- `llvm/MC/MCDecoder.h`
- `llvm/MC/MCDecoderOps.h`
- `llvm/MC/MCDisassembler/MCDisassembler.h`
- `llvm/MC/MCInst.h`
- `llvm/MC/MCInstrInfo.h`
- `llvm/MC/MCSubtargetInfo.h`
- `llvm/MC/TargetRegistry.h`
- `llvm/Support/Compiler.h`
- `llvm/Support/Endian.h`
- `LoongArchGenDisassemblerTables.inc`
