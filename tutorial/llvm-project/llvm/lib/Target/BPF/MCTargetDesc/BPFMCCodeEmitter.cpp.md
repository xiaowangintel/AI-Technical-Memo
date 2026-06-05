# BPFMCCodeEmitter.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/BPF/MCTargetDesc/BPFMCCodeEmitter.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file implements the BPFMCCodeEmitter class.
- 目的（中文）: 将 MC 指令编码为目标专用的二进制机器码。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- BPFMCCodeEmitter.cpp - Convert BPF code to machine code -----------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file implements the BPFMCCodeEmitter class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "MCTargetDesc/BPFMCFixups.h"
  14: #include "MCTargetDesc/BPFMCTargetDesc.h"
  15: #include "llvm/ADT/SmallVector.h"
  16: #include "llvm/MC/MCCodeEmitter.h"
  17: #include "llvm/MC/MCContext.h"
  18: #include "llvm/MC/MCExpr.h"
  19: #include "llvm/MC/MCFixup.h"
  20: #include "llvm/MC/MCInst.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

### Lines 21-40

```cpp
  21: #include "llvm/MC/MCInstrInfo.h"
  22: #include "llvm/MC/MCRegisterInfo.h"
  23: #include "llvm/MC/MCSubtargetInfo.h"
  24: #include "llvm/Support/EndianStream.h"
  25: #include <cassert>
  26: #include <cstdint>
  27: 
  28: using namespace llvm;
  29: 
  30: #define DEBUG_TYPE "mccodeemitter"
  31: 
  32: namespace {
  33: 
  34: class BPFMCCodeEmitter : public MCCodeEmitter {
  35:   const MCRegisterInfo &MRI;
  36:   bool IsLittleEndian;
  37:   MCContext &Ctx;
  38: 
  39: public:
  40:   BPFMCCodeEmitter(const MCInstrInfo &, const MCRegisterInfo &mri,
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as BPFMCCodeEmitter, which organize the target-specific behavior exposed by the file.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 BPFMCCodeEmitter 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 41-60

```cpp
  41:                    bool IsLittleEndian, MCContext &ctx)
  42:       : MRI(mri), IsLittleEndian(IsLittleEndian), Ctx(ctx) {}
  43:   BPFMCCodeEmitter(const BPFMCCodeEmitter &) = delete;
  44:   void operator=(const BPFMCCodeEmitter &) = delete;
  45:   ~BPFMCCodeEmitter() override = default;
  46: 
  47:   // getBinaryCodeForInstr - TableGen'erated function for getting the
  48:   // binary encoding for an instruction.
  49:   uint64_t getBinaryCodeForInstr(const MCInst &MI,
  50:                                  SmallVectorImpl<MCFixup> &Fixups,
  51:                                  const MCSubtargetInfo &STI) const;
  52: 
  53:   // getMachineOpValue - Return binary encoding of operand. If the machin
  54:   // operand requires relocation, record the relocation and return zero.
  55:   unsigned getMachineOpValue(const MCInst &MI, const MCOperand &MO,
  56:                              SmallVectorImpl<MCFixup> &Fixups,
  57:                              const MCSubtargetInfo &STI) const;
  58: 
  59:   uint64_t getMemoryOpValue(const MCInst &MI, unsigned Op,
  60:                             SmallVectorImpl<MCFixup> &Fixups,
```

- EN: Function bodies or method definitions such as MRI contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: MRI 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 61-80

```cpp
  61:                             const MCSubtargetInfo &STI) const;
  62: 
  63:   void encodeInstruction(const MCInst &MI, SmallVectorImpl<char> &CB,
  64:                          SmallVectorImpl<MCFixup> &Fixups,
  65:                          const MCSubtargetInfo &STI) const override;
  66: };
  67: 
  68: } // end anonymous namespace
  69: 
  70: MCCodeEmitter *llvm::createBPFMCCodeEmitter(const MCInstrInfo &MCII,
  71:                                             MCContext &Ctx) {
  72:   return new BPFMCCodeEmitter(MCII, *Ctx.getRegisterInfo(), true, Ctx);
  73: }
  74: 
  75: MCCodeEmitter *llvm::createBPFbeMCCodeEmitter(const MCInstrInfo &MCII,
  76:                                               MCContext &Ctx) {
  77:   return new BPFMCCodeEmitter(MCII, *Ctx.getRegisterInfo(), false, Ctx);
  78: }
  79: 
  80: static void addFixup(SmallVectorImpl<MCFixup> &Fixups, uint32_t Offset,
```

- EN: This range continues the implementation of the backend component described by BPFMCCodeEmitter.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 81-100

```cpp
  81:                      const MCExpr *Value, uint16_t Kind, bool PCRel = false) {
  82:   Fixups.push_back(MCFixup::create(Offset, Value, Kind, PCRel));
  83: }
  84: 
  85: unsigned BPFMCCodeEmitter::getMachineOpValue(const MCInst &MI,
  86:                                              const MCOperand &MO,
  87:                                              SmallVectorImpl<MCFixup> &Fixups,
  88:                                              const MCSubtargetInfo &STI) const {
  89:   if (MO.isReg())
  90:     return MRI.getEncodingValue(MO.getReg());
  91:   if (MO.isImm()) {
  92:     uint64_t Imm = MO.getImm();
  93:     uint64_t High32Bits = Imm >> 32, High33Bits = Imm >> 31;
  94:     if (MI.getOpcode() != BPF::LD_imm64 && High32Bits != 0 &&
  95:         High33Bits != 0x1FFFFFFFFULL) {
  96:       Ctx.reportWarning(MI.getLoc(),
  97:                         "immediate out of range, shall fit in 32 bits");
  98:     }
  99:     return static_cast<unsigned>(Imm);
 100:   }
```

- EN: Function bodies or method definitions such as getMachineOpValue contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: getMachineOpValue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 101-120

```cpp
 101: 
 102:   assert(MO.isExpr());
 103: 
 104:   const MCExpr *Expr = MO.getExpr();
 105: 
 106:   assert(Expr->getKind() == MCExpr::SymbolRef);
 107: 
 108:   if (MI.getOpcode() == BPF::JAL)
 109:     // func call name
 110:     addFixup(Fixups, 0, Expr, FK_Data_4, true);
 111:   else if (MI.getOpcode() == BPF::LD_imm64)
 112:     addFixup(Fixups, 0, Expr, FK_SecRel_8);
 113:   else if (MI.getOpcode() == BPF::JMPL)
 114:     addFixup(Fixups, 0, Expr, BPF::FK_BPF_PCRel_4, true);
 115:   else
 116:     // bb label
 117:     addFixup(Fixups, 0, Expr, FK_Data_2, true);
 118: 
 119:   return 0;
 120: }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 121-140

```cpp
 121: 
 122: static uint8_t SwapBits(uint8_t Val)
 123: {
 124:   return (Val & 0x0F) << 4 | (Val & 0xF0) >> 4;
 125: }
 126: 
 127: void BPFMCCodeEmitter::encodeInstruction(const MCInst &MI,
 128:                                          SmallVectorImpl<char> &CB,
 129:                                          SmallVectorImpl<MCFixup> &Fixups,
 130:                                          const MCSubtargetInfo &STI) const {
 131:   unsigned Opcode = MI.getOpcode();
 132:   raw_svector_ostream OS(CB);
 133:   support::endian::Writer OSE(OS, IsLittleEndian ? llvm::endianness::little
 134:                                                  : llvm::endianness::big);
 135: 
 136:   if (Opcode == BPF::LD_imm64 || Opcode == BPF::LD_pseudo) {
 137:     uint64_t Value = getBinaryCodeForInstr(MI, Fixups, STI);
 138:     CB.push_back(Value >> 56);
 139:     if (IsLittleEndian)
 140:       CB.push_back((Value >> 48) & 0xff);
```

- EN: Function bodies or method definitions such as SwapBits, encodeInstruction contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: SwapBits, encodeInstruction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 141-160

```cpp
 141:     else
 142:       CB.push_back(SwapBits((Value >> 48) & 0xff));
 143:     OSE.write<uint16_t>(0);
 144:     OSE.write<uint32_t>(Value & 0xffffFFFF);
 145: 
 146:     const MCOperand &MO = MI.getOperand(1);
 147:     uint64_t Imm = MO.isImm() ? MO.getImm() : 0;
 148:     OSE.write<uint8_t>(0);
 149:     OSE.write<uint8_t>(0);
 150:     OSE.write<uint16_t>(0);
 151:     OSE.write<uint32_t>(Imm >> 32);
 152:   } else {
 153:     // Get instruction encoding and emit it
 154:     uint64_t Value = getBinaryCodeForInstr(MI, Fixups, STI);
 155:     CB.push_back(Value >> 56);
 156:     if (IsLittleEndian)
 157:       CB.push_back(char((Value >> 48) & 0xff));
 158:     else
 159:       CB.push_back(SwapBits((Value >> 48) & 0xff));
 160:     OSE.write<uint16_t>((Value >> 32) & 0xffff);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 161-180

```cpp
 161:     OSE.write<uint32_t>(Value & 0xffffFFFF);
 162:   }
 163: }
 164: 
 165: // Encode BPF Memory Operand
 166: uint64_t BPFMCCodeEmitter::getMemoryOpValue(const MCInst &MI, unsigned Op,
 167:                                             SmallVectorImpl<MCFixup> &Fixups,
 168:                                             const MCSubtargetInfo &STI) const {
 169:   // For CMPXCHG instructions, output is implicitly in R0/W0,
 170:   // so memory operand starts from operand 0.
 171:   int MemOpStartIndex = 1, Opcode = MI.getOpcode();
 172:   if (Opcode == BPF::CMPXCHGW32 || Opcode == BPF::CMPXCHGD)
 173:     MemOpStartIndex = 0;
 174: 
 175:   uint64_t Encoding;
 176:   const MCOperand Op1 = MI.getOperand(MemOpStartIndex);
 177:   assert(Op1.isReg() && "First operand is not register.");
 178:   Encoding = MRI.getEncodingValue(Op1.getReg());
 179:   Encoding <<= 16;
 180:   MCOperand Op2 = MI.getOperand(MemOpStartIndex + 1);
```

- EN: Function bodies or method definitions such as getMemoryOpValue contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: getMemoryOpValue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 181-186

```cpp
 181:   assert(Op2.isImm() && "Second operand is not immediate.");
 182:   Encoding |= Op2.getImm() & 0xffff;
 183:   return Encoding;
 184: }
 185: 
 186: #include "BPFGenMCCodeEmitter.inc"
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

## Key Concepts / 关键概念

- MC instruction representation / MC 指令表示
- Binary encoding / 二进制编码
- CPU feature modelling / CPU 特性建模

## Dependencies / 依赖关系

- Direct includes / 直接包含: `MCTargetDesc/BPFMCFixups.h`, `MCTargetDesc/BPFMCTargetDesc.h`, `llvm/ADT/SmallVector.h`, `llvm/MC/MCCodeEmitter.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCFixup.h`, `llvm/MC/MCInst.h`
- LLVM subsystems / LLVM 子系统: LLVM MC
- Generated or companion files / 生成或配套文件: `BPFGenMCCodeEmitter.inc`
