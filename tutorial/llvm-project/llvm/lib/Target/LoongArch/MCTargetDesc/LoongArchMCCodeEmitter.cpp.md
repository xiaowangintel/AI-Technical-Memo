# LoongArchMCCodeEmitter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/MCTargetDesc/LoongArchMCCodeEmitter.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责将后端连接到 LLVM MC 层描述符。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //=- LoongArchMCCodeEmitter.cpp - Convert LoongArch code to machine code --===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file implements the LoongArchMCCodeEmitter class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "LoongArchFixupKinds.h"
  14: #include "MCTargetDesc/LoongArchMCAsmInfo.h"
  15: #include "MCTargetDesc/LoongArchMCTargetDesc.h"
  16: #include "llvm/BinaryFormat/ELF.h"
  17: #include "llvm/MC/MCCodeEmitter.h"
  18: #include "llvm/MC/MCContext.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `LoongArchFixupKinds.h`, `LoongArchMCAsmInfo.h`, `LoongArchMCTargetDesc.h`, `ELF.h`, `MCCodeEmitter.h`, `MCContext.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `LoongArchFixupKinds.h`, `LoongArchMCAsmInfo.h`, `LoongArchMCTargetDesc.h`, `ELF.h`, `MCCodeEmitter.h`, `MCContext.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "llvm/MC/MCInstBuilder.h"
  20: #include "llvm/MC/MCInstrInfo.h"
  21: #include "llvm/MC/MCRegisterInfo.h"
  22: #include "llvm/MC/MCSubtargetInfo.h"
  23: #include "llvm/Support/Casting.h"
  24: #include "llvm/Support/EndianStream.h"
  25: 
  26: using namespace llvm;
  27: 
  28: #define DEBUG_TYPE "mccodeemitter"
  29: 
  30: namespace {
  31: class LoongArchMCCodeEmitter : public MCCodeEmitter {
  32:   LoongArchMCCodeEmitter(const LoongArchMCCodeEmitter &) = delete;
  33:   void operator=(const LoongArchMCCodeEmitter &) = delete;
  34:   MCContext &Ctx;
  35:   MCInstrInfo const &MCII;
  36: 
```
- **EN**: It imports dependencies such as `MCInstBuilder.h`, `MCInstrInfo.h`, `MCRegisterInfo.h`, `MCSubtargetInfo.h`, `Casting.h`, `EndianStream.h` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `LoongArchMCCodeEmitter`. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `MCInstBuilder.h`, `MCInstrInfo.h`, `MCRegisterInfo.h`, `MCSubtargetInfo.h`, `Casting.h`, `EndianStream.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `LoongArchMCCodeEmitter` 等 TableGen 记录。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 37-54 / 第 37-54 行
```cpp
  37: public:
  38:   LoongArchMCCodeEmitter(MCContext &ctx, MCInstrInfo const &MCII)
  39:       : Ctx(ctx), MCII(MCII) {}
  40: 
  41:   ~LoongArchMCCodeEmitter() override = default;
  42: 
  43:   void encodeInstruction(const MCInst &MI, SmallVectorImpl<char> &CB,
  44:                          SmallVectorImpl<MCFixup> &Fixups,
  45:                          const MCSubtargetInfo &STI) const override;
  46: 
  47:   template <unsigned Opc>
  48:   void expandToVectorLDI(const MCInst &MI, SmallVectorImpl<char> &CB,
  49:                          SmallVectorImpl<MCFixup> &Fixups,
  50:                          const MCSubtargetInfo &STI) const;
  51: 
  52:   void expandAddTPRel(const MCInst &MI, SmallVectorImpl<char> &CB,
  53:                       SmallVectorImpl<MCFixup> &Fixups,
  54:                       const MCSubtargetInfo &STI) const;
```
- **EN**: The range implements or declares functions including `LoongArchMCCodeEmitter`.
- **CN**: 这一段实现或声明了 `LoongArchMCCodeEmitter` 等函数。

### Lines 55-72 / 第 55-72 行
```cpp
  55: 
  56:   /// TableGen'erated function for getting the binary encoding for an
  57:   /// instruction.
  58:   uint64_t getBinaryCodeForInstr(const MCInst &MI,
  59:                                  SmallVectorImpl<MCFixup> &Fixups,
  60:                                  const MCSubtargetInfo &STI) const;
  61: 
  62:   /// Return binary encoding of operand. If the machine operand requires
  63:   /// relocation, record the relocation and return zero.
  64:   unsigned getMachineOpValue(const MCInst &MI, const MCOperand &MO,
  65:                              SmallVectorImpl<MCFixup> &Fixups,
  66:                              const MCSubtargetInfo &STI) const;
  67: 
  68:   /// Return binary encoding of an immediate operand specified by OpNo.
  69:   /// The value returned is the value of the immediate minus 1.
  70:   /// Note that this function is dedicated to specific immediate types,
  71:   /// e.g. uimm2_plus1.
  72:   unsigned getImmOpValueSub1(const MCInst &MI, unsigned OpNo,
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 73-90 / 第 73-90 行
```cpp
  73:                              SmallVectorImpl<MCFixup> &Fixups,
  74:                              const MCSubtargetInfo &STI) const;
  75: 
  76:   /// Return binary encoding of an immediate operand specified by OpNo.
  77:   /// The value returned is the value of the immediate shifted right
  78:   //  arithmetically by N.
  79:   /// Note that this function is dedicated to specific immediate types,
  80:   /// e.g. simm14_lsl2, simm16_lsl2, simm21_lsl2 and simm26_lsl2.
  81:   template <unsigned N>
  82:   unsigned getImmOpValueAsr(const MCInst &MI, unsigned OpNo,
  83:                             SmallVectorImpl<MCFixup> &Fixups,
  84:                             const MCSubtargetInfo &STI) const {
  85:     const MCOperand &MO = MI.getOperand(OpNo);
  86:     if (MO.isImm()) {
  87:       unsigned Res = MI.getOperand(OpNo).getImm();
  88:       assert((Res & ((1U << N) - 1U)) == 0 && "lowest N bits are non-zero");
  89:       return Res >> N;
  90:     }
```
- **EN**: The range implements or declares functions including `getImmOpValueAsr`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `getImmOpValueAsr` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 91-108 / 第 91-108 行
```cpp
  91:     return getExprOpValue(MI, MO, Fixups, STI);
  92:   }
  93: 
  94:   unsigned getExprOpValue(const MCInst &MI, const MCOperand &MO,
  95:                           SmallVectorImpl<MCFixup> &Fixups,
  96:                           const MCSubtargetInfo &STI) const;
  97: };
  98: } // end namespace
  99: 
 100: static void addFixup(SmallVectorImpl<MCFixup> &Fixups, uint32_t Offset,
 101:                      const MCExpr *Value, uint16_t Kind) {
 102:   bool PCRel = false;
 103:   switch (Kind) {
 104:   case LoongArch::fixup_loongarch_b16:
 105:   case LoongArch::fixup_loongarch_b21:
 106:   case LoongArch::fixup_loongarch_b26:
 107:     PCRel = true;
 108:   }
```
- **EN**: The range implements or declares functions including `addFixup`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `addFixup` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 109-126 / 第 109-126 行
```cpp
 109:   Fixups.push_back(MCFixup::create(Offset, Value, Kind, PCRel));
 110: }
 111: 
 112: unsigned
 113: LoongArchMCCodeEmitter::getMachineOpValue(const MCInst &MI, const MCOperand &MO,
 114:                                           SmallVectorImpl<MCFixup> &Fixups,
 115:                                           const MCSubtargetInfo &STI) const {
 116: 
 117:   if (MO.isReg())
 118:     return Ctx.getRegisterInfo()->getEncodingValue(MO.getReg());
 119: 
 120:   if (MO.isImm())
 121:     return static_cast<unsigned>(MO.getImm());
 122: 
 123:   // MO must be an Expr.
 124:   assert(MO.isExpr());
 125:   return getExprOpValue(MI, MO, Fixups, STI);
 126: }
```
- **EN**: The range implements or declares functions including `LoongArchMCCodeEmitter::getMachineOpValue`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchMCCodeEmitter::getMachineOpValue` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 127-144 / 第 127-144 行
```cpp
 127: 
 128: unsigned
 129: LoongArchMCCodeEmitter::getImmOpValueSub1(const MCInst &MI, unsigned OpNo,
 130:                                           SmallVectorImpl<MCFixup> &Fixups,
 131:                                           const MCSubtargetInfo &STI) const {
 132:   return MI.getOperand(OpNo).getImm() - 1;
 133: }
 134: 
 135: unsigned
 136: LoongArchMCCodeEmitter::getExprOpValue(const MCInst &MI, const MCOperand &MO,
 137:                                        SmallVectorImpl<MCFixup> &Fixups,
 138:                                        const MCSubtargetInfo &STI) const {
 139:   assert(MO.isExpr() && "getExprOpValue expects only expressions");
 140:   bool RelaxCandidate = false;
 141:   bool EnableRelax = STI.hasFeature(LoongArch::FeatureRelax);
 142:   const MCExpr *Expr = MO.getExpr();
 143:   MCExpr::ExprKind Kind = Expr->getKind();
 144:   unsigned FixupKind = LoongArch::fixup_loongarch_invalid;
```
- **EN**: The range implements or declares functions including `LoongArchMCCodeEmitter::getImmOpValueSub1`, `LoongArchMCCodeEmitter::getExprOpValue`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchMCCodeEmitter::getImmOpValueSub1`, `LoongArchMCCodeEmitter::getExprOpValue` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 145-162 / 第 145-162 行
```cpp
 145:   if (Kind == MCExpr::Specifier) {
 146:     const LoongArchMCExpr *LAExpr = cast<LoongArchMCExpr>(Expr);
 147:     FixupKind = LAExpr->getSpecifier();
 148:     RelaxCandidate = LAExpr->getRelaxHint();
 149:     switch (uint16_t(LAExpr->getSpecifier())) {
 150:     case LoongArchMCExpr::VK_None:
 151:       llvm_unreachable("Unhandled fixup kind!");
 152:     case ELF::R_LARCH_TLS_LE_ADD_R:
 153:       llvm_unreachable("ELF::R_LARCH_TLS_LE_ADD_R should not represent an "
 154:                        "instruction operand");
 155:     case ELF::R_LARCH_B16:
 156:       FixupKind = LoongArch::fixup_loongarch_b16;
 157:       break;
 158:     case ELF::R_LARCH_B21:
 159:       FixupKind = LoongArch::fixup_loongarch_b21;
 160:       break;
 161:     case ELF::R_LARCH_B26:
 162:       FixupKind = LoongArch::fixup_loongarch_b26;
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 163-180 / 第 163-180 行
```cpp
 163:       break;
 164:     case ELF::R_LARCH_MARK_LA:
 165:       // Match gas behavior: generate `R_LARCH_MARK_LA` relocation when using
 166:       // `la.abs`.
 167:       Fixups.push_back(
 168:           MCFixup::create(0, MCConstantExpr::create(0, Ctx),
 169:                           FirstLiteralRelocationKind + ELF::R_LARCH_MARK_LA));
 170:       [[fallthrough]];
 171:     case ELF::R_LARCH_ABS_HI20:
 172:       FixupKind = LoongArch::fixup_loongarch_abs_hi20;
 173:       break;
 174:     case ELF::R_LARCH_ABS_LO12:
 175:       FixupKind = LoongArch::fixup_loongarch_abs_lo12;
 176:       break;
 177:     case ELF::R_LARCH_ABS64_LO20:
 178:       FixupKind = LoongArch::fixup_loongarch_abs64_lo20;
 179:       break;
 180:     case ELF::R_LARCH_ABS64_HI12:
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 181-198 / 第 181-198 行
```cpp
 181:       FixupKind = LoongArch::fixup_loongarch_abs64_hi12;
 182:       break;
 183:     case ELF::R_LARCH_CALL30:
 184:     case ELF::R_LARCH_CALL36:
 185:     case ELF::R_LARCH_TLS_LE_HI20_R:
 186:     case ELF::R_LARCH_TLS_LE_LO12_R:
 187:       RelaxCandidate = true;
 188:       break;
 189:     }
 190:   } else if (Kind == MCExpr::SymbolRef) {
 191:     switch (MI.getOpcode()) {
 192:     default:
 193:       break;
 194:     case LoongArch::BEQ:
 195:     case LoongArch::BNE:
 196:     case LoongArch::BLT:
 197:     case LoongArch::BGE:
 198:     case LoongArch::BLTU:
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 199-216 / 第 199-216 行
```cpp
 199:     case LoongArch::BGEU:
 200:       FixupKind = LoongArch::fixup_loongarch_b16;
 201:       break;
 202:     case LoongArch::BEQZ:
 203:     case LoongArch::BNEZ:
 204:     case LoongArch::BCEQZ:
 205:     case LoongArch::BCNEZ:
 206:       FixupKind = LoongArch::fixup_loongarch_b21;
 207:       break;
 208:     case LoongArch::B:
 209:     case LoongArch::BL:
 210:       FixupKind = LoongArch::fixup_loongarch_b26;
 211:       break;
 212:     }
 213:   }
 214: 
 215:   assert(FixupKind != LoongArch::fixup_loongarch_invalid &&
 216:          "Unhandled expression!");
```
- **EN**: The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 217-234 / 第 217-234 行
```cpp
 217: 
 218:   addFixup(Fixups, 0, Expr, FixupKind);
 219:   // If linker relaxation is enabled and supported by this relocation, set
 220:   // a bit so that if fixup is unresolved, a R_LARCH_RELAX relocation will be
 221:   // appended.
 222:   if (EnableRelax && RelaxCandidate)
 223:     Fixups.back().setLinkerRelaxable();
 224: 
 225:   return 0;
 226: }
 227: 
 228: template <unsigned Opc>
 229: void LoongArchMCCodeEmitter::expandToVectorLDI(
 230:     const MCInst &MI, SmallVectorImpl<char> &CB,
 231:     SmallVectorImpl<MCFixup> &Fixups, const MCSubtargetInfo &STI) const {
 232:   int64_t Imm = MI.getOperand(1).getImm() & 0x3FF;
 233:   switch (MI.getOpcode()) {
 234:   case LoongArch::PseudoVREPLI_B:
```
- **EN**: The range implements or declares functions including `LoongArchMCCodeEmitter::expandToVectorLDI`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchMCCodeEmitter::expandToVectorLDI` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 235-252 / 第 235-252 行
```cpp
 235:   case LoongArch::PseudoXVREPLI_B:
 236:     break;
 237:   case LoongArch::PseudoVREPLI_H:
 238:   case LoongArch::PseudoXVREPLI_H:
 239:     Imm |= 0x400;
 240:     break;
 241:   case LoongArch::PseudoVREPLI_W:
 242:   case LoongArch::PseudoXVREPLI_W:
 243:     Imm |= 0x800;
 244:     break;
 245:   case LoongArch::PseudoVREPLI_D:
 246:   case LoongArch::PseudoXVREPLI_D:
 247:     Imm |= 0xC00;
 248:     break;
 249:   }
 250:   MCInst TmpInst = MCInstBuilder(Opc).addOperand(MI.getOperand(0)).addImm(Imm);
 251:   uint32_t Binary = getBinaryCodeForInstr(TmpInst, Fixups, STI);
 252:   support::endian::write(CB, Binary, llvm::endianness::little);
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 253-270 / 第 253-270 行
```cpp
 253: }
 254: 
 255: void LoongArchMCCodeEmitter::expandAddTPRel(const MCInst &MI,
 256:                                             SmallVectorImpl<char> &CB,
 257:                                             SmallVectorImpl<MCFixup> &Fixups,
 258:                                             const MCSubtargetInfo &STI) const {
 259:   MCOperand Rd = MI.getOperand(0);
 260:   MCOperand Rj = MI.getOperand(1);
 261:   MCOperand Rk = MI.getOperand(2);
 262:   MCOperand Symbol = MI.getOperand(3);
 263:   assert(Symbol.isExpr() &&
 264:          "Expected expression as third input to TP-relative add");
 265: 
 266:   const LoongArchMCExpr *Expr = dyn_cast<LoongArchMCExpr>(Symbol.getExpr());
 267:   assert(Expr && Expr->getSpecifier() == ELF::R_LARCH_TLS_LE_ADD_R &&
 268:          "Expected %le_add_r relocation on TP-relative symbol");
 269: 
 270:   // Emit the correct %le_add_r relocation for the symbol.
```
- **EN**: The range implements or declares functions including `LoongArchMCCodeEmitter::expandAddTPRel`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchMCCodeEmitter::expandAddTPRel` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 271-288 / 第 271-288 行
```cpp
 271:   addFixup(Fixups, 0, Expr, ELF::R_LARCH_TLS_LE_ADD_R);
 272:   if (STI.hasFeature(LoongArch::FeatureRelax))
 273:     Fixups.back().setLinkerRelaxable();
 274: 
 275:   // Emit a normal ADD instruction with the given operands.
 276:   unsigned ADD = MI.getOpcode() == LoongArch::PseudoAddTPRel_D
 277:                      ? LoongArch::ADD_D
 278:                      : LoongArch::ADD_W;
 279:   MCInst TmpInst =
 280:       MCInstBuilder(ADD).addOperand(Rd).addOperand(Rj).addOperand(Rk);
 281:   uint32_t Binary = getBinaryCodeForInstr(TmpInst, Fixups, STI);
 282:   support::endian::write(CB, Binary, llvm::endianness::little);
 283: }
 284: 
 285: void LoongArchMCCodeEmitter::encodeInstruction(
 286:     const MCInst &MI, SmallVectorImpl<char> &CB,
 287:     SmallVectorImpl<MCFixup> &Fixups, const MCSubtargetInfo &STI) const {
 288:   const MCInstrDesc &Desc = MCII.get(MI.getOpcode());
```
- **EN**: The range implements or declares functions including `LoongArchMCCodeEmitter::encodeInstruction`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchMCCodeEmitter::encodeInstruction` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 289-306 / 第 289-306 行
```cpp
 289:   // Get byte count of instruction.
 290:   unsigned Size = Desc.getSize();
 291: 
 292:   switch (MI.getOpcode()) {
 293:   default:
 294:     break;
 295:   case LoongArch::PseudoVREPLI_B:
 296:   case LoongArch::PseudoVREPLI_H:
 297:   case LoongArch::PseudoVREPLI_W:
 298:   case LoongArch::PseudoVREPLI_D:
 299:     return expandToVectorLDI<LoongArch::VLDI>(MI, CB, Fixups, STI);
 300:   case LoongArch::PseudoXVREPLI_B:
 301:   case LoongArch::PseudoXVREPLI_H:
 302:   case LoongArch::PseudoXVREPLI_W:
 303:   case LoongArch::PseudoXVREPLI_D:
 304:     return expandToVectorLDI<LoongArch::XVLDI>(MI, CB, Fixups, STI);
 305:   case LoongArch::PseudoAddTPRel_W:
 306:   case LoongArch::PseudoAddTPRel_D:
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 307-324 / 第 307-324 行
```cpp
 307:     return expandAddTPRel(MI, CB, Fixups, STI);
 308:   }
 309: 
 310:   switch (Size) {
 311:   default:
 312:     llvm_unreachable("Unhandled encodeInstruction length!");
 313:   case 4: {
 314:     uint32_t Bits = getBinaryCodeForInstr(MI, Fixups, STI);
 315:     support::endian::write(CB, Bits, llvm::endianness::little);
 316:     break;
 317:   }
 318:   }
 319: }
 320: 
 321: MCCodeEmitter *llvm::createLoongArchMCCodeEmitter(const MCInstrInfo &MCII,
 322:                                                   MCContext &Ctx) {
 323:   return new LoongArchMCCodeEmitter(Ctx, MCII);
 324: }
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 325-326 / 第 325-326 行
```cpp
 325: 
 326: #include "LoongArchGenMCCodeEmitter.inc"
```
- **EN**: It imports dependencies such as `LoongArchGenMCCodeEmitter.inc` that expose the LLVM and target interfaces used in later logic.
- **CN**: 它引入了 `LoongArchGenMCCodeEmitter.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `LoongArchFixupKinds.h`
- `MCTargetDesc/LoongArchMCAsmInfo.h`
- `MCTargetDesc/LoongArchMCTargetDesc.h`
- `llvm/BinaryFormat/ELF.h`
- `llvm/MC/MCCodeEmitter.h`
- `llvm/MC/MCContext.h`
- `llvm/MC/MCInstBuilder.h`
- `llvm/MC/MCInstrInfo.h`
- `llvm/MC/MCRegisterInfo.h`
- `llvm/MC/MCSubtargetInfo.h`
- `llvm/Support/Casting.h`
- `llvm/Support/EndianStream.h`
- `LoongArchGenMCCodeEmitter.inc`
