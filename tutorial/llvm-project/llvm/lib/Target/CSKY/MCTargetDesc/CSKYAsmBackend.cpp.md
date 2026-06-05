# CSKYAsmBackend.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/MCTargetDesc/CSKYAsmBackend.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Implements the MC assembler backend, including fixups, relaxation, and object emission policies.
- 目的（中文）: 实现 MC 汇编后端，包括重定位修正、指令松弛和目标文件输出策略。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- CSKYAsmBackend.cpp - CSKY Assembler Backend -----------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "CSKYAsmBackend.h"
  10: #include "MCTargetDesc/CSKYMCTargetDesc.h"
  11: #include "llvm/ADT/DenseMap.h"
  12: #include "llvm/MC/MCAssembler.h"
  13: #include "llvm/MC/MCContext.h"
  14: #include "llvm/MC/MCObjectWriter.h"
  15: #include "llvm/MC/MCValue.h"
  16: #include "llvm/Support/Debug.h"
  17: 
  18: #define DEBUG_TYPE "csky-asmbackend"
  19: 
  20: using namespace llvm;
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。

### Lines 21-40

```cpp
  21: 
  22: std::unique_ptr<MCObjectTargetWriter>
  23: CSKYAsmBackend::createObjectTargetWriter() const {
  24:   return createCSKYELFObjectWriter();
  25: }
  26: 
  27: MCFixupKindInfo CSKYAsmBackend::getFixupKindInfo(MCFixupKind Kind) const {
  28: 
  29:   static llvm::DenseMap<unsigned, MCFixupKindInfo> Infos = {
  30:       {CSKY::Fixups::fixup_csky_addr32, {"fixup_csky_addr32", 0, 32, 0}},
  31:       {CSKY::Fixups::fixup_csky_addr_hi16, {"fixup_csky_addr_hi16", 0, 32, 0}},
  32:       {CSKY::Fixups::fixup_csky_addr_lo16, {"fixup_csky_addr_lo16", 0, 32, 0}},
  33:       {CSKY::Fixups::fixup_csky_pcrel_imm16_scale2,
  34:        {"fixup_csky_pcrel_imm16_scale2", 0, 32, 0}},
  35:       {CSKY::Fixups::fixup_csky_pcrel_uimm16_scale4,
  36:        {"fixup_csky_pcrel_uimm16_scale4", 0, 32, 0}},
  37:       {CSKY::Fixups::fixup_csky_pcrel_uimm8_scale4,
  38:        {"fixup_csky_pcrel_uimm8_scale4", 0, 32, 0}},
  39:       {CSKY::Fixups::fixup_csky_pcrel_imm26_scale2,
  40:        {"fixup_csky_pcrel_imm26_scale2", 0, 32, 0}},
```

- EN: Function bodies or method definitions such as createObjectTargetWriter, getFixupKindInfo contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: createObjectTargetWriter, getFixupKindInfo 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 41-60

```cpp
  41:       {CSKY::Fixups::fixup_csky_pcrel_imm18_scale2,
  42:        {"fixup_csky_pcrel_imm18_scale2", 0, 32, 0}},
  43:       {CSKY::Fixups::fixup_csky_got32, {"fixup_csky_got32", 0, 32, 0}},
  44:       {CSKY::Fixups::fixup_csky_got_imm18_scale4,
  45:        {"fixup_csky_got_imm18_scale4", 0, 32, 0}},
  46:       {CSKY::Fixups::fixup_csky_gotoff, {"fixup_csky_gotoff", 0, 32, 0}},
  47:       {CSKY::Fixups::fixup_csky_gotpc, {"fixup_csky_gotpc", 0, 32, 0}},
  48:       {CSKY::Fixups::fixup_csky_plt32, {"fixup_csky_plt32", 0, 32, 0}},
  49:       {CSKY::Fixups::fixup_csky_plt_imm18_scale4,
  50:        {"fixup_csky_plt_imm18_scale4", 0, 32, 0}},
  51:       {CSKY::Fixups::fixup_csky_pcrel_imm10_scale2,
  52:        {"fixup_csky_pcrel_imm10_scale2", 0, 16, 0}},
  53:       {CSKY::Fixups::fixup_csky_pcrel_uimm7_scale4,
  54:        {"fixup_csky_pcrel_uimm7_scale4", 0, 16, 0}},
  55:       {CSKY::Fixups::fixup_csky_doffset_imm18,
  56:        {"fixup_csky_doffset_imm18", 0, 18, 0}},
  57:       {CSKY::Fixups::fixup_csky_doffset_imm18_scale2,
  58:        {"fixup_csky_doffset_imm18_scale2", 0, 18, 0}},
  59:       {CSKY::Fixups::fixup_csky_doffset_imm18_scale4,
  60:        {"fixup_csky_doffset_imm18_scale4", 0, 18, 0}}};
```

- EN: This range continues the implementation of the backend component described by CSKYAsmBackend.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 61-80

```cpp
  61: 
  62:   assert(Infos.size() == CSKY::NumTargetFixupKinds &&
  63:          "Not all fixup kinds added to Infos array");
  64: 
  65:   if (mc::isRelocation(Kind))
  66:     return {};
  67:   if (Kind < FirstTargetFixupKind)
  68:     return MCAsmBackend::getFixupKindInfo(Kind);
  69:   return Infos[Kind];
  70: }
  71: 
  72: static uint64_t adjustFixupValue(const MCFixup &Fixup, uint64_t Value,
  73:                                  MCContext &Ctx) {
  74:   switch (Fixup.getKind()) {
  75:   default:
  76:     llvm_unreachable("Unknown fixup kind!");
  77:   case CSKY::fixup_csky_got32:
  78:   case CSKY::fixup_csky_got_imm18_scale4:
  79:   case CSKY::fixup_csky_gotoff:
  80:   case CSKY::fixup_csky_gotpc:
```

- EN: Function bodies or method definitions such as adjustFixupValue contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: adjustFixupValue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 81-100

```cpp
  81:   case CSKY::fixup_csky_plt32:
  82:   case CSKY::fixup_csky_plt_imm18_scale4:
  83:     llvm_unreachable("Relocation should be unconditionally forced\n");
  84:   case FK_Data_1:
  85:   case FK_Data_2:
  86:   case FK_Data_4:
  87:   case FK_Data_8:
  88:     return Value;
  89:   case CSKY::fixup_csky_addr32:
  90:     return Value & 0xffffffff;
  91:   case CSKY::fixup_csky_pcrel_imm16_scale2:
  92:     if (!isIntN(17, Value))
  93:       Ctx.reportError(Fixup.getLoc(), "out of range pc-relative fixup value.");
  94:     if (Value & 0x1)
  95:       Ctx.reportError(Fixup.getLoc(), "fixup value must be 2-byte aligned.");
  96: 
  97:     return (Value >> 1) & 0xffff;
  98:   case CSKY::fixup_csky_pcrel_uimm16_scale4:
  99:     if (!isUIntN(18, Value))
 100:       Ctx.reportError(Fixup.getLoc(), "out of range pc-relative fixup value.");
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 101-120

```cpp
 101:     if (Value & 0x3)
 102:       Ctx.reportError(Fixup.getLoc(), "fixup value must be 4-byte aligned.");
 103: 
 104:     return (Value >> 2) & 0xffff;
 105:   case CSKY::fixup_csky_pcrel_imm26_scale2:
 106:     if (!isIntN(27, Value))
 107:       Ctx.reportError(Fixup.getLoc(), "out of range pc-relative fixup value.");
 108:     if (Value & 0x1)
 109:       Ctx.reportError(Fixup.getLoc(), "fixup value must be 2-byte aligned.");
 110: 
 111:     return (Value >> 1) & 0x3ffffff;
 112:   case CSKY::fixup_csky_pcrel_imm18_scale2:
 113:     if (!isIntN(19, Value))
 114:       Ctx.reportError(Fixup.getLoc(), "out of range pc-relative fixup value.");
 115:     if (Value & 0x1)
 116:       Ctx.reportError(Fixup.getLoc(), "fixup value must be 2-byte aligned.");
 117: 
 118:     return (Value >> 1) & 0x3ffff;
 119:   case CSKY::fixup_csky_pcrel_uimm8_scale4: {
 120:     if (!isUIntN(10, Value))
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 121-140

```cpp
 121:       Ctx.reportError(Fixup.getLoc(), "out of range pc-relative fixup value.");
 122:     if (Value & 0x3)
 123:       Ctx.reportError(Fixup.getLoc(), "fixup value must be 4-byte aligned.");
 124: 
 125:     unsigned IMM4L = (Value >> 2) & 0xf;
 126:     unsigned IMM4H = (Value >> 6) & 0xf;
 127: 
 128:     Value = (IMM4H << 21) | (IMM4L << 4);
 129:     return Value;
 130:   }
 131:   case CSKY::fixup_csky_pcrel_imm10_scale2:
 132:     if (!isIntN(11, Value))
 133:       Ctx.reportError(Fixup.getLoc(), "out of range pc-relative fixup value.");
 134:     if (Value & 0x1)
 135:       Ctx.reportError(Fixup.getLoc(), "fixup value must be 2-byte aligned.");
 136: 
 137:     return (Value >> 1) & 0x3ff;
 138:   case CSKY::fixup_csky_pcrel_uimm7_scale4:
 139:     if ((Value >> 2) > 0xfe)
 140:       Ctx.reportError(Fixup.getLoc(), "out of range pc-relative fixup value.");
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 141-160

```cpp
 141:     if (Value & 0x3)
 142:       Ctx.reportError(Fixup.getLoc(), "fixup value must be 4-byte aligned.");
 143: 
 144:     if ((Value >> 2) <= 0x7f) {
 145:       unsigned IMM5L = (Value >> 2) & 0x1f;
 146:       unsigned IMM2H = (Value >> 7) & 0x3;
 147: 
 148:       Value = (1 << 12) | (IMM2H << 8) | IMM5L;
 149:     } else {
 150:       unsigned IMM5L = (~Value >> 2) & 0x1f;
 151:       unsigned IMM2H = (~Value >> 7) & 0x3;
 152: 
 153:       Value = (IMM2H << 8) | IMM5L;
 154:     }
 155: 
 156:     return Value;
 157:   }
 158: }
 159: 
 160: bool CSKYAsmBackend::fixupNeedsRelaxationAdvanced(const MCFragment &,
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 161-180

```cpp
 161:                                                   const MCFixup &Fixup,
 162:                                                   const MCValue &,
 163:                                                   uint64_t Value,
 164:                                                   bool Resolved) const {
 165:   // Return true if the symbol is unresolved.
 166:   if (!Resolved)
 167:     return true;
 168: 
 169:   int64_t Offset = int64_t(Value);
 170:   switch (Fixup.getKind()) {
 171:   default:
 172:     return false;
 173:   case CSKY::fixup_csky_pcrel_imm10_scale2:
 174:     return !isShiftedInt<10, 1>(Offset);
 175:   case CSKY::fixup_csky_pcrel_imm16_scale2:
 176:     return !isShiftedInt<16, 1>(Offset);
 177:   case CSKY::fixup_csky_pcrel_imm26_scale2:
 178:     return !isShiftedInt<26, 1>(Offset);
 179:   case CSKY::fixup_csky_pcrel_uimm7_scale4:
 180:     return ((Value >> 2) > 0xfe) || (Value & 0x3);
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 181-200

```cpp
 181:   }
 182: }
 183: 
 184: std::optional<bool> CSKYAsmBackend::evaluateFixup(const MCFragment &F,
 185:                                                   MCFixup &Fixup, MCValue &,
 186:                                                   uint64_t &Value) {
 187:   // These forward-only constant pool load fixups compute displacement relative
 188:   // to AlignDown(PC, 4). Pre-seed Value with the low bits so the generic
 189:   // evaluateFixup effectively subtracts the aligned offset. Subtract Stretch
 190:   // to use the pre-Stretch (old) source offset, avoiding an epoch mismatch
 191:   // with the generic Stretch-adjusted displacement (see ARMAsmBackend).
 192:   switch (Fixup.getKind()) {
 193:   case CSKY::fixup_csky_pcrel_uimm16_scale4:
 194:   case CSKY::fixup_csky_pcrel_uimm8_scale4:
 195:   case CSKY::fixup_csky_pcrel_uimm7_scale4:
 196:     Value =
 197:         (Asm->getFragmentOffset(F) - Asm->getStretch() + Fixup.getOffset()) % 4;
 198:   }
 199:   return {};
 200: }
```

- EN: Function bodies or method definitions such as evaluateFixup contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: evaluateFixup 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 201-220

```cpp
 201: 
 202: void CSKYAsmBackend::applyFixup(const MCFragment &F, const MCFixup &Fixup,
 203:                                 const MCValue &Target, uint8_t *Data,
 204:                                 uint64_t Value, bool IsResolved) {
 205:   if (IsResolved && shouldForceRelocation(Fixup, Target))
 206:     IsResolved = false;
 207:   maybeAddReloc(F, Fixup, Target, Value, IsResolved);
 208: 
 209:   MCFixupKind Kind = Fixup.getKind();
 210:   if (mc::isRelocation(Kind))
 211:     return;
 212:   MCContext &Ctx = getContext();
 213:   MCFixupKindInfo Info = getFixupKindInfo(Kind);
 214:   if (!Value)
 215:     return; // Doesn't change encoding.
 216:   // Apply any target-specific value adjustments.
 217:   Value = adjustFixupValue(Fixup, Value, Ctx);
 218: 
 219:   // Shift the value into position.
 220:   Value <<= Info.TargetOffset;
```

- EN: Function bodies or method definitions such as applyFixup contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: applyFixup 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 221-240

```cpp
 221: 
 222:   unsigned NumBytes = alignTo(Info.TargetSize + Info.TargetOffset, 8) / 8;
 223: 
 224:   assert(Fixup.getOffset() + NumBytes <= F.getSize() &&
 225:          "Invalid fixup offset!");
 226: 
 227:   // For each byte of the fragment that the fixup touches, mask in the
 228:   // bits from the fixup value.
 229:   bool IsLittleEndian = (Endian == llvm::endianness::little);
 230:   bool IsInstFixup = (Kind >= FirstTargetFixupKind);
 231: 
 232:   if (IsLittleEndian && IsInstFixup && (NumBytes == 4)) {
 233:     Data[0] |= uint8_t((Value >> 16) & 0xff);
 234:     Data[1] |= uint8_t((Value >> 24) & 0xff);
 235:     Data[2] |= uint8_t(Value & 0xff);
 236:     Data[3] |= uint8_t((Value >> 8) & 0xff);
 237:   } else {
 238:     for (unsigned I = 0; I != NumBytes; I++) {
 239:       unsigned Idx = IsLittleEndian ? I : (NumBytes - 1 - I);
 240:       Data[Idx] |= uint8_t((Value >> (I * 8)) & 0xff);
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 241-260

```cpp
 241:     }
 242:   }
 243: }
 244: 
 245: bool CSKYAsmBackend::mayNeedRelaxation(unsigned Opcode, ArrayRef<MCOperand>,
 246:                                        const MCSubtargetInfo &STI) const {
 247:   switch (Opcode) {
 248:   default:
 249:     return false;
 250:   case CSKY::JBR32:
 251:   case CSKY::JBT32:
 252:   case CSKY::JBF32:
 253:   case CSKY::JBSR32:
 254:     if (!STI.hasFeature(CSKY::Has2E3))
 255:       return false;
 256:     return true;
 257:   case CSKY::JBR16:
 258:   case CSKY::JBT16:
 259:   case CSKY::JBF16:
 260:   case CSKY::LRW16:
```

- EN: Function bodies or method definitions such as mayNeedRelaxation contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: mayNeedRelaxation 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 261-280

```cpp
 261:   case CSKY::BR16:
 262:     return true;
 263:   }
 264: }
 265: 
 266: bool CSKYAsmBackend::shouldForceRelocation(const MCFixup &Fixup,
 267:                                            const MCValue &Target /*STI*/) {
 268:   if (Target.getSpecifier())
 269:     return true;
 270:   switch (Fixup.getKind()) {
 271:   default:
 272:     break;
 273:   case CSKY::fixup_csky_doffset_imm18:
 274:   case CSKY::fixup_csky_doffset_imm18_scale2:
 275:   case CSKY::fixup_csky_doffset_imm18_scale4:
 276:     return true;
 277:   }
 278: 
 279:   return false;
 280: }
```

- EN: Function bodies or method definitions such as shouldForceRelocation contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: shouldForceRelocation 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 281-300

```cpp
 281: 
 282: void CSKYAsmBackend::relaxInstruction(MCInst &Inst,
 283:                                       const MCSubtargetInfo &STI) const {
 284:   MCInst Res;
 285: 
 286:   switch (Inst.getOpcode()) {
 287:   default:
 288:     LLVM_DEBUG(Inst.dump());
 289:     llvm_unreachable("Opcode not expected!");
 290:   case CSKY::LRW16:
 291:     Res.setOpcode(CSKY::LRW32);
 292:     Res.addOperand(Inst.getOperand(0));
 293:     Res.addOperand(Inst.getOperand(1));
 294:     break;
 295:   case CSKY::BR16:
 296:     Res.setOpcode(CSKY::BR32);
 297:     Res.addOperand(Inst.getOperand(0));
 298:     break;
 299:   case CSKY::JBSR32:
 300:     Res.setOpcode(CSKY::JSRI32);
```

- EN: Function bodies or method definitions such as relaxInstruction contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: relaxInstruction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 301-320

```cpp
 301:     Res.addOperand(Inst.getOperand(1));
 302:     break;
 303:   case CSKY::JBR32:
 304:     Res.setOpcode(CSKY::JMPI32);
 305:     Res.addOperand(Inst.getOperand(1));
 306:     break;
 307:   case CSKY::JBT32:
 308:   case CSKY::JBF32:
 309:     Res.setOpcode(Inst.getOpcode() == CSKY::JBT32 ? CSKY::JBT_E : CSKY::JBF_E);
 310:     Res.addOperand(Inst.getOperand(0));
 311:     Res.addOperand(Inst.getOperand(1));
 312:     Res.addOperand(Inst.getOperand(2));
 313:     break;
 314:   case CSKY::JBR16:
 315:     Res.setOpcode(CSKY::JBR32);
 316:     Res.addOperand(Inst.getOperand(0));
 317:     Res.addOperand(Inst.getOperand(1));
 318:     break;
 319:   case CSKY::JBT16:
 320:   case CSKY::JBF16:
```

- EN: This range continues the implementation of the backend component described by CSKYAsmBackend.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 321-340

```cpp
 321:     // ck801
 322:     unsigned opcode;
 323:     if (STI.hasFeature(CSKY::HasE2))
 324:       opcode = Inst.getOpcode() == CSKY::JBT16 ? CSKY::JBT32 : CSKY::JBF32;
 325:     else
 326:       opcode = Inst.getOpcode() == CSKY::JBT16 ? CSKY::JBT_E : CSKY::JBF_E;
 327: 
 328:     Res.setOpcode(opcode);
 329:     Res.addOperand(Inst.getOperand(0));
 330:     Res.addOperand(Inst.getOperand(1));
 331:     Res.addOperand(Inst.getOperand(2));
 332:     break;
 333:   }
 334:   Inst = std::move(Res);
 335: }
 336: 
 337: bool CSKYAsmBackend::writeNopData(raw_ostream &OS, uint64_t Count,
 338:                                   const MCSubtargetInfo *STI) const {
 339:   OS.write_zeros(Count);
 340:   return true;
```

- EN: Function bodies or method definitions such as writeNopData contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: writeNopData 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 341-348

```cpp
 341: }
 342: 
 343: MCAsmBackend *llvm::createCSKYAsmBackend(const Target &T,
 344:                                          const MCSubtargetInfo &STI,
 345:                                          const MCRegisterInfo &MRI,
 346:                                          const MCTargetOptions &Options) {
 347:   return new CSKYAsmBackend(STI, Options);
 348: }
```

- EN: This range continues the implementation of the backend component described by CSKYAsmBackend.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

## Key Concepts / 关键概念

- Fixups and relaxation / 修正与松弛
- Object emission rules / 目标文件输出规则
- MC instruction representation / MC 指令表示
- Assembler backend policies / 汇编后端策略
- CPU feature modelling / CPU 特性建模
- ELF object support / ELF 目标文件支持

## Dependencies / 依赖关系

- Direct includes / 直接包含: `CSKYAsmBackend.h`, `MCTargetDesc/CSKYMCTargetDesc.h`, `llvm/ADT/DenseMap.h`, `llvm/MC/MCAssembler.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCObjectWriter.h`, `llvm/MC/MCValue.h`, `llvm/Support/Debug.h`
- LLVM subsystems / LLVM 子系统: LLVM MC
- Local companions / 本地配套文件: `CSKYAsmBackend.h`
