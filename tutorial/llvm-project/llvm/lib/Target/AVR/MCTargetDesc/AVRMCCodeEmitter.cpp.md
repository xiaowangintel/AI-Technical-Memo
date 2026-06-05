# AVRMCCodeEmitter.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/AVR/MCTargetDesc/AVRMCCodeEmitter.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file implements the AVRMCCodeEmitter class.
- 目的（中文）: 将 MC 指令编码为目标专用的二进制机器码。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- AVRMCCodeEmitter.cpp - Convert AVR Code to Machine Code -----------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file implements the AVRMCCodeEmitter class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "AVRMCCodeEmitter.h"
  14: 
  15: #include "MCTargetDesc/AVRMCAsmInfo.h"
  16: #include "MCTargetDesc/AVRMCTargetDesc.h"
  17: 
  18: #include "llvm/ADT/APFloat.h"
  19: #include "llvm/ADT/SmallVector.h"
  20: #include "llvm/MC/MCContext.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

### Lines 21-40

```cpp
  21: #include "llvm/MC/MCExpr.h"
  22: #include "llvm/MC/MCFixup.h"
  23: #include "llvm/MC/MCInst.h"
  24: #include "llvm/MC/MCInstrInfo.h"
  25: #include "llvm/MC/MCRegisterInfo.h"
  26: #include "llvm/MC/MCSubtargetInfo.h"
  27: #include "llvm/Support/Casting.h"
  28: #include "llvm/Support/EndianStream.h"
  29: 
  30: #define DEBUG_TYPE "mccodeemitter"
  31: 
  32: #define GET_INSTRMAP_INFO
  33: #include "AVRGenInstrInfo.inc"
  34: #undef GET_INSTRMAP_INFO
  35: 
  36: namespace llvm {
  37: 
  38: static void addFixup(SmallVectorImpl<MCFixup> &Fixups, uint32_t Offset,
  39:                      const MCExpr *Value, uint16_t Kind) {
  40:   bool PCRel = false;
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as addFixup contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 addFixup 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 41-60

```cpp
  41:   switch (Kind) {
  42:   case AVR::fixup_7_pcrel:
  43:   case AVR::fixup_13_pcrel:
  44:     PCRel = true;
  45:   }
  46:   Fixups.push_back(MCFixup::create(Offset, Value, Kind, PCRel));
  47: }
  48: 
  49: /// Performs a post-encoding step on a `LD` or `ST` instruction.
  50: ///
  51: /// The encoding of the LD/ST family of instructions is inconsistent w.r.t
  52: /// the pointer register and the addressing mode.
  53: ///
  54: /// The permutations of the format are as followed:
  55: /// ld Rd, X    `1001 000d dddd 1100`
  56: /// ld Rd, X+   `1001 000d dddd 1101`
  57: /// ld Rd, -X   `1001 000d dddd 1110`
  58: ///
  59: /// ld Rd, Y    `1000 000d dddd 1000`
  60: /// ld Rd, Y+   `1001 000d dddd 1001`
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 61-80

```cpp
  61: /// ld Rd, -Y   `1001 000d dddd 1010`
  62: ///
  63: /// ld Rd, Z    `1000 000d dddd 0000`
  64: /// ld Rd, Z+   `1001 000d dddd 0001`
  65: /// ld Rd, -Z   `1001 000d dddd 0010`
  66: ///                 ^
  67: ///                 |
  68: /// Note this one inconsistent bit - it is 1 sometimes and 0 at other times.
  69: /// There is no logical pattern. Looking at a truth table, the following
  70: /// formula can be derived to fit the pattern:
  71: //
  72: /// ```
  73: /// inconsistent_bit = is_predec OR is_postinc OR is_reg_x
  74: /// ```
  75: //
  76: /// We manually set this bit in this post encoder method.
  77: unsigned
  78: AVRMCCodeEmitter::loadStorePostEncoder(const MCInst &MI, unsigned EncodedValue,
  79:                                        const MCSubtargetInfo &STI) const {
  80: 
```

- EN: Function bodies or method definitions such as loadStorePostEncoder contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: loadStorePostEncoder 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 81-100

```cpp
  81:   assert(MI.getOperand(0).isReg() && MI.getOperand(1).isReg() &&
  82:          "the load/store operands must be registers");
  83: 
  84:   unsigned Opcode = MI.getOpcode();
  85: 
  86:   // Get the index of the pointer register operand.
  87:   unsigned Idx = 0;
  88:   if (Opcode == AVR::LDRdPtrPd || Opcode == AVR::LDRdPtrPi ||
  89:       Opcode == AVR::LDRdPtr)
  90:     Idx = 1;
  91: 
  92:   // Check if we need to set the inconsistent bit
  93:   bool IsPredec = Opcode == AVR::LDRdPtrPd || Opcode == AVR::STPtrPdRr;
  94:   bool IsPostinc = Opcode == AVR::LDRdPtrPi || Opcode == AVR::STPtrPiRr;
  95:   if (MI.getOperand(Idx).getReg() == AVR::R27R26 || IsPredec || IsPostinc)
  96:     EncodedValue |= (1 << 12);
  97: 
  98:   // Encode the pointer register.
  99:   switch (MI.getOperand(Idx).getReg().id()) {
 100:   case AVR::R27R26:
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 101-120

```cpp
 101:     EncodedValue |= 0xc;
 102:     break;
 103:   case AVR::R29R28:
 104:     EncodedValue |= 0x8;
 105:     break;
 106:   case AVR::R31R30:
 107:     break;
 108:   default:
 109:     llvm_unreachable("invalid pointer register");
 110:     break;
 111:   }
 112: 
 113:   return EncodedValue;
 114: }
 115: 
 116: template <AVR::Fixups Fixup>
 117: unsigned
 118: AVRMCCodeEmitter::encodeRelCondBrTarget(const MCInst &MI, unsigned OpNo,
 119:                                         SmallVectorImpl<MCFixup> &Fixups,
 120:                                         const MCSubtargetInfo &STI) const {
```

- EN: Function bodies or method definitions such as encodeRelCondBrTarget contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: encodeRelCondBrTarget 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 121-140

```cpp
 121:   const MCOperand &MO = MI.getOperand(OpNo);
 122: 
 123:   if (MO.isExpr()) {
 124:     addFixup(Fixups, 0, MO.getExpr(), MCFixupKind(Fixup));
 125:     return 0;
 126:   }
 127: 
 128:   assert(MO.isImm());
 129: 
 130:   // Take the size of the current instruction away.
 131:   // With labels, this is implicitly done.
 132:   auto target = MO.getImm();
 133:   AVR::fixups::adjustBranchTarget(target);
 134:   return target;
 135: }
 136: 
 137: /// Encodes a `memri` operand.
 138: /// The operand is 7-bits.
 139: /// * The lower 6 bits is the immediate
 140: /// * The upper bit is the pointer register bit (Z=0,Y=1)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 141-160

```cpp
 141: unsigned AVRMCCodeEmitter::encodeMemri(const MCInst &MI, unsigned OpNo,
 142:                                        SmallVectorImpl<MCFixup> &Fixups,
 143:                                        const MCSubtargetInfo &STI) const {
 144:   auto RegOp = MI.getOperand(OpNo);
 145:   auto OffsetOp = MI.getOperand(OpNo + 1);
 146: 
 147:   assert(RegOp.isReg() && "Expected register operand");
 148: 
 149:   uint8_t RegBit = 0;
 150: 
 151:   switch (RegOp.getReg().id()) {
 152:   default:
 153:     Ctx.reportError(MI.getLoc(), "Expected either Y or Z register");
 154:     return 0;
 155:   case AVR::R31R30:
 156:     RegBit = 0;
 157:     break; // Z register
 158:   case AVR::R29R28:
 159:     RegBit = 1;
 160:     break; // Y register
```

- EN: Function bodies or method definitions such as encodeMemri contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: encodeMemri 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 161-180

```cpp
 161:   }
 162: 
 163:   int8_t OffsetBits;
 164: 
 165:   if (OffsetOp.isImm()) {
 166:     OffsetBits = OffsetOp.getImm();
 167:   } else if (OffsetOp.isExpr()) {
 168:     OffsetBits = 0;
 169:     addFixup(Fixups, 0, OffsetOp.getExpr(), AVR::fixup_6);
 170:   } else {
 171:     llvm_unreachable("Invalid value for offset");
 172:   }
 173: 
 174:   return (RegBit << 6) | OffsetBits;
 175: }
 176: 
 177: unsigned AVRMCCodeEmitter::encodeComplement(const MCInst &MI, unsigned OpNo,
 178:                                             SmallVectorImpl<MCFixup> &Fixups,
 179:                                             const MCSubtargetInfo &STI) const {
 180:   // The operand should be an immediate.
```

- EN: Function bodies or method definitions such as encodeComplement contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: encodeComplement 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 181-200

```cpp
 181:   assert(MI.getOperand(OpNo).isImm());
 182: 
 183:   auto Imm = MI.getOperand(OpNo).getImm();
 184:   return (~0) - Imm;
 185: }
 186: 
 187: template <AVR::Fixups Fixup, unsigned Offset>
 188: unsigned AVRMCCodeEmitter::encodeImm(const MCInst &MI, unsigned OpNo,
 189:                                      SmallVectorImpl<MCFixup> &Fixups,
 190:                                      const MCSubtargetInfo &STI) const {
 191:   auto MO = MI.getOperand(OpNo);
 192: 
 193:   if (MO.isExpr()) {
 194:     if (isa<AVRMCExpr>(MO.getExpr())) {
 195:       // If the expression is already an AVRMCExpr (i.e. a lo8(symbol),
 196:       // we shouldn't perform any more fixups. Without this check, we would
 197:       // instead create a fixup to the symbol named 'lo8(symbol)' which
 198:       // is not correct.
 199:       return getExprOpValue(MO.getExpr(), Fixups, STI);
 200:     }
```

- EN: Function bodies or method definitions such as encodeImm contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: encodeImm 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 201-220

```cpp
 201: 
 202:     MCFixupKind FixupKind = static_cast<MCFixupKind>(Fixup);
 203:     addFixup(Fixups, Offset, MO.getExpr(), FixupKind);
 204: 
 205:     return 0;
 206:   }
 207: 
 208:   assert(MO.isImm());
 209:   return MO.getImm();
 210: }
 211: 
 212: unsigned AVRMCCodeEmitter::encodeCallTarget(const MCInst &MI, unsigned OpNo,
 213:                                             SmallVectorImpl<MCFixup> &Fixups,
 214:                                             const MCSubtargetInfo &STI) const {
 215:   auto MO = MI.getOperand(OpNo);
 216: 
 217:   if (MO.isExpr()) {
 218:     MCFixupKind FixupKind = AVR::fixup_call;
 219:     addFixup(Fixups, 0, MO.getExpr(), FixupKind);
 220:     return 0;
```

- EN: Function bodies or method definitions such as encodeCallTarget contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: encodeCallTarget 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 221-240

```cpp
 221:   }
 222: 
 223:   assert(MO.isImm());
 224: 
 225:   auto Target = MO.getImm();
 226:   AVR::fixups::adjustBranchTarget(Target);
 227:   return Target;
 228: }
 229: 
 230: unsigned AVRMCCodeEmitter::getExprOpValue(const MCExpr *Expr,
 231:                                           SmallVectorImpl<MCFixup> &Fixups,
 232:                                           const MCSubtargetInfo &STI) const {
 233: 
 234:   MCExpr::ExprKind Kind = Expr->getKind();
 235: 
 236:   if (Kind == MCExpr::Binary) {
 237:     Expr = static_cast<const MCBinaryExpr *>(Expr)->getLHS();
 238:     Kind = Expr->getKind();
 239:   }
 240: 
```

- EN: Function bodies or method definitions such as getExprOpValue contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: getExprOpValue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 241-260

```cpp
 241:   if (Kind == MCExpr::Specifier) {
 242:     AVRMCExpr const *AVRExpr = cast<AVRMCExpr>(Expr);
 243:     int64_t Result;
 244:     if (AVRExpr->evaluateAsConstant(Result)) {
 245:       return Result;
 246:     }
 247: 
 248:     MCFixupKind FixupKind = static_cast<MCFixupKind>(AVRExpr->getFixupKind());
 249:     addFixup(Fixups, 0, AVRExpr, FixupKind);
 250:     return 0;
 251:   }
 252: 
 253:   assert(Kind == MCExpr::SymbolRef);
 254:   return 0;
 255: }
 256: 
 257: unsigned AVRMCCodeEmitter::getMachineOpValue(const MCInst &MI,
 258:                                              const MCOperand &MO,
 259:                                              SmallVectorImpl<MCFixup> &Fixups,
 260:                                              const MCSubtargetInfo &STI) const {
```

- EN: Function bodies or method definitions such as getMachineOpValue contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: getMachineOpValue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 261-280

```cpp
 261:   if (MO.isReg())
 262:     return Ctx.getRegisterInfo()->getEncodingValue(MO.getReg());
 263:   if (MO.isImm())
 264:     return static_cast<unsigned>(MO.getImm());
 265: 
 266:   if (MO.isDFPImm())
 267:     return static_cast<unsigned>(bit_cast<double>(MO.getDFPImm()));
 268: 
 269:   // MO must be an Expr.
 270:   assert(MO.isExpr());
 271: 
 272:   return getExprOpValue(MO.getExpr(), Fixups, STI);
 273: }
 274: 
 275: void AVRMCCodeEmitter::encodeInstruction(const MCInst &MI,
 276:                                          SmallVectorImpl<char> &CB,
 277:                                          SmallVectorImpl<MCFixup> &Fixups,
 278:                                          const MCSubtargetInfo &STI) const {
 279:   const MCInstrDesc &Desc = MCII.get(MI.getOpcode());
 280: 
```

- EN: Function bodies or method definitions such as encodeInstruction contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: encodeInstruction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 281-300

```cpp
 281:   // Get byte count of instruction
 282:   unsigned Size = Desc.getSize();
 283: 
 284:   assert(Size > 0 && "Instruction size cannot be zero");
 285: 
 286:   uint64_t BinaryOpCode = getBinaryCodeForInstr(MI, Fixups, STI);
 287: 
 288:   for (int64_t i = Size / 2 - 1; i >= 0; --i) {
 289:     uint16_t Word = (BinaryOpCode >> (i * 16)) & 0xFFFF;
 290:     support::endian::write(CB, Word, llvm::endianness::little);
 291:   }
 292: }
 293: 
 294: MCCodeEmitter *createAVRMCCodeEmitter(const MCInstrInfo &MCII, MCContext &Ctx) {
 295:   return new AVRMCCodeEmitter(MCII, Ctx);
 296: }
 297: 
 298: #include "AVRGenMCCodeEmitter.inc"
 299: 
 300: } // end of namespace llvm
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

## Key Concepts / 关键概念

- MC instruction representation / MC 指令表示
- Binary encoding / 二进制编码
- CPU feature modelling / CPU 特性建模

## Dependencies / 依赖关系

- Direct includes / 直接包含: `AVRMCCodeEmitter.h`, `MCTargetDesc/AVRMCAsmInfo.h`, `MCTargetDesc/AVRMCTargetDesc.h`, `llvm/ADT/APFloat.h`, `llvm/ADT/SmallVector.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCFixup.h`
- LLVM subsystems / LLVM 子系统: LLVM MC
- Generated or companion files / 生成或配套文件: `AVRGenInstrInfo.inc`, `AVRGenMCCodeEmitter.inc`
- Local companions / 本地配套文件: `AVRMCCodeEmitter.h`
