# LoongArchAsmBackend.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/MCTargetDesc/LoongArchAsmBackend.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责将后端连接到 LLVM MC 层描述符。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```cpp
   1: //===-- LoongArchAsmBackend.cpp - LoongArch Assembler Backend -*- C++ -*---===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file implements the LoongArchAsmBackend class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "LoongArchAsmBackend.h"
  14: #include "LoongArchFixupKinds.h"
  15: #include "llvm/BinaryFormat/ELF.h"
  16: #include "llvm/MC/MCAsmInfo.h"
  17: #include "llvm/MC/MCAssembler.h"
  18: #include "llvm/MC/MCContext.h"
  19: #include "llvm/MC/MCELFObjectWriter.h"
  20: #include "llvm/MC/MCExpr.h"
  21: #include "llvm/MC/MCSection.h"
  22: #include "llvm/MC/MCValue.h"
  23: #include "llvm/Support/EndianStream.h"
  24: #include "llvm/Support/LEB128.h"
  25: #include "llvm/Support/MathExtras.h"
  26: 
  27: #define DEBUG_TYPE "loongarch-asmbackend"
  28: 
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `LoongArchAsmBackend.h`, `LoongArchFixupKinds.h`, `ELF.h`, `MCAsmInfo.h`, `MCAssembler.h`, `MCContext.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `LoongArchAsmBackend.h`, `LoongArchFixupKinds.h`, `ELF.h`, `MCAsmInfo.h`, `MCAssembler.h`, `MCContext.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 29-56 / 第 29-56 行
```cpp
  29: using namespace llvm;
  30: 
  31: LoongArchAsmBackend::LoongArchAsmBackend(const MCSubtargetInfo &STI,
  32:                                          uint8_t OSABI, bool Is64Bit,
  33:                                          const MCTargetOptions &Options)
  34:     : MCAsmBackend(llvm::endianness::little), STI(STI), OSABI(OSABI),
  35:       Is64Bit(Is64Bit), TargetOptions(Options) {}
  36: 
  37: std::optional<MCFixupKind>
  38: LoongArchAsmBackend::getFixupKind(StringRef Name) const {
  39:   if (STI.getTargetTriple().isOSBinFormatELF()) {
  40:     auto Type = llvm::StringSwitch<unsigned>(Name)
  41: #define ELF_RELOC(X, Y) .Case(#X, Y)
  42: #include "llvm/BinaryFormat/ELFRelocs/LoongArch.def"
  43: #undef ELF_RELOC
  44:                     .Case("BFD_RELOC_NONE", ELF::R_LARCH_NONE)
  45:                     .Case("BFD_RELOC_32", ELF::R_LARCH_32)
  46:                     .Case("BFD_RELOC_64", ELF::R_LARCH_64)
  47:                     .Default(-1u);
  48:     if (Type != -1u)
  49:       return static_cast<MCFixupKind>(FirstLiteralRelocationKind + Type);
  50:   }
  51:   return std::nullopt;
  52: }
  53: 
  54: MCFixupKindInfo LoongArchAsmBackend::getFixupKindInfo(MCFixupKind Kind) const {
  55:   const static MCFixupKindInfo Infos[] = {
  56:       // This table *must* be in the order that the fixup_* kinds are defined in
```
- **EN**: It imports dependencies such as `LoongArch.def` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `MCAsmBackend`, `LoongArchAsmBackend::getFixupKindInfo`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `LoongArch.def` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `MCAsmBackend`, `LoongArchAsmBackend::getFixupKindInfo` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 57-84 / 第 57-84 行
```cpp
  57:       // LoongArchFixupKinds.h.
  58:       //
  59:       // {name, offset, bits, flags}
  60:       {"fixup_loongarch_b16", 10, 16, 0},
  61:       {"fixup_loongarch_b21", 0, 26, 0},
  62:       {"fixup_loongarch_b26", 0, 26, 0},
  63:       {"fixup_loongarch_abs_hi20", 5, 20, 0},
  64:       {"fixup_loongarch_abs_lo12", 10, 12, 0},
  65:       {"fixup_loongarch_abs64_lo20", 5, 20, 0},
  66:       {"fixup_loongarch_abs64_hi12", 10, 12, 0},
  67:   };
  68: 
  69:   static_assert((std::size(Infos)) == LoongArch::NumTargetFixupKinds,
  70:                 "Not all fixup kinds added to Infos array");
  71: 
  72:   // Fixup kinds from .reloc directive are like R_LARCH_NONE. They
  73:   // do not require any extra processing.
  74:   if (mc::isRelocation(Kind))
  75:     return {};
  76: 
  77:   if (Kind < FirstTargetFixupKind)
  78:     return MCAsmBackend::getFixupKindInfo(Kind);
  79: 
  80:   assert(unsigned(Kind - FirstTargetFixupKind) <
  81:              LoongArch::NumTargetFixupKinds &&
  82:          "Invalid kind!");
  83:   return Infos[Kind - FirstTargetFixupKind];
  84: }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 85-112 / 第 85-112 行
```cpp
  85: 
  86: static void reportOutOfRangeError(MCContext &Ctx, SMLoc Loc, unsigned N) {
  87:   Ctx.reportError(Loc, "fixup value out of range [" + Twine(llvm::minIntN(N)) +
  88:                            ", " + Twine(llvm::maxIntN(N)) + "]");
  89: }
  90: 
  91: static uint64_t adjustFixupValue(const MCFixup &Fixup, uint64_t Value,
  92:                                  MCContext &Ctx) {
  93:   switch (Fixup.getKind()) {
  94:   default:
  95:     llvm_unreachable("Unknown fixup kind");
  96:   case FK_Data_1:
  97:   case FK_Data_2:
  98:   case FK_Data_4:
  99:   case FK_Data_8:
 100:   case FK_Data_leb128:
 101:     return Value;
 102:   case LoongArch::fixup_loongarch_b16: {
 103:     if (!isInt<18>(Value))
 104:       reportOutOfRangeError(Ctx, Fixup.getLoc(), 18);
 105:     if (Value % 4)
 106:       Ctx.reportError(Fixup.getLoc(), "fixup value must be 4-byte aligned");
 107:     return (Value >> 2) & 0xffff;
 108:   }
 109:   case LoongArch::fixup_loongarch_b21: {
 110:     if (!isInt<23>(Value))
 111:       reportOutOfRangeError(Ctx, Fixup.getLoc(), 23);
 112:     if (Value % 4)
```
- **EN**: The range implements or declares functions including `reportOutOfRangeError`, `adjustFixupValue`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `reportOutOfRangeError`, `adjustFixupValue` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 113-140 / 第 113-140 行
```cpp
 113:       Ctx.reportError(Fixup.getLoc(), "fixup value must be 4-byte aligned");
 114:     return ((Value & 0x3fffc) << 8) | ((Value >> 18) & 0x1f);
 115:   }
 116:   case LoongArch::fixup_loongarch_b26: {
 117:     if (!isInt<28>(Value))
 118:       reportOutOfRangeError(Ctx, Fixup.getLoc(), 28);
 119:     if (Value % 4)
 120:       Ctx.reportError(Fixup.getLoc(), "fixup value must be 4-byte aligned");
 121:     return ((Value & 0x3fffc) << 8) | ((Value >> 18) & 0x3ff);
 122:   }
 123:   case LoongArch::fixup_loongarch_abs_hi20:
 124:     return (Value >> 12) & 0xfffff;
 125:   case LoongArch::fixup_loongarch_abs_lo12:
 126:     return Value & 0xfff;
 127:   case LoongArch::fixup_loongarch_abs64_lo20:
 128:     return (Value >> 32) & 0xfffff;
 129:   case LoongArch::fixup_loongarch_abs64_hi12:
 130:     return (Value >> 52) & 0xfff;
 131:   }
 132: }
 133: 
 134: static void fixupLeb128(MCContext &Ctx, const MCFixup &Fixup, uint8_t *Data,
 135:                         uint64_t Value) {
 136:   unsigned I;
 137:   for (I = 0; Value; ++I, Value >>= 7)
 138:     Data[I] |= uint8_t(Value & 0x7f);
 139: }
 140: 
```
- **EN**: The range implements or declares functions including `fixupLeb128`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `fixupLeb128` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 141-168 / 第 141-168 行
```cpp
 141: void LoongArchAsmBackend::applyFixup(const MCFragment &F, const MCFixup &Fixup,
 142:                                      const MCValue &Target, uint8_t *Data,
 143:                                      uint64_t Value, bool IsResolved) {
 144:   IsResolved = addReloc(F, Fixup, Target, Value, IsResolved);
 145:   if (!Value)
 146:     return; // Doesn't change encoding.
 147: 
 148:   auto Kind = Fixup.getKind();
 149:   if (mc::isRelocation(Kind))
 150:     return;
 151:   MCFixupKindInfo Info = getFixupKindInfo(Kind);
 152:   MCContext &Ctx = getContext();
 153: 
 154:   // Fixup leb128 separately.
 155:   if (Fixup.getKind() == FK_Data_leb128)
 156:     return fixupLeb128(Ctx, Fixup, Data, Value);
 157: 
 158:   // Apply any target-specific value adjustments.
 159:   Value = adjustFixupValue(Fixup, Value, Ctx);
 160: 
 161:   // Shift the value into position.
 162:   Value <<= Info.TargetOffset;
 163: 
 164:   unsigned NumBytes = alignTo(Info.TargetSize + Info.TargetOffset, 8) / 8;
 165: 
 166:   assert(Fixup.getOffset() + NumBytes <= F.getSize() &&
 167:          "Invalid fixup offset!");
 168:   // For each byte of the fragment that the fixup touches, mask in the
```
- **EN**: The range implements or declares functions including `LoongArchAsmBackend::applyFixup`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchAsmBackend::applyFixup` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 169-196 / 第 169-196 行
```cpp
 169:   // bits from the fixup value.
 170:   for (unsigned I = 0; I != NumBytes; ++I) {
 171:     Data[I] |= uint8_t((Value >> (I * 8)) & 0xff);
 172:   }
 173: }
 174: 
 175: static inline std::pair<MCFixupKind, MCFixupKind>
 176: getRelocPairForSize(unsigned Size) {
 177:   switch (Size) {
 178:   default:
 179:     llvm_unreachable("unsupported fixup size");
 180:   case 6:
 181:     return std::make_pair(ELF::R_LARCH_ADD6, ELF::R_LARCH_SUB6);
 182:   case 8:
 183:     return std::make_pair(ELF::R_LARCH_ADD8, ELF::R_LARCH_SUB8);
 184:   case 16:
 185:     return std::make_pair(ELF::R_LARCH_ADD16, ELF::R_LARCH_SUB16);
 186:   case 32:
 187:     return std::make_pair(ELF::R_LARCH_ADD32, ELF::R_LARCH_SUB32);
 188:   case 64:
 189:     return std::make_pair(ELF::R_LARCH_ADD64, ELF::R_LARCH_SUB64);
 190:   case 128:
 191:     return std::make_pair(ELF::R_LARCH_ADD_ULEB128, ELF::R_LARCH_SUB_ULEB128);
 192:   }
 193: }
 194: 
 195: // Check if an R_LARCH_ALIGN relocation is needed for an alignment directive.
 196: // If conditions are met, compute the padding size and create a fixup encoding
```
- **EN**: The range implements or declares functions including `getRelocPairForSize`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `getRelocPairForSize` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 197-224 / 第 197-224 行
```cpp
 197: // the padding size in the addend. If MaxBytesToEmit is smaller than the padding
 198: // size, the fixup encodes MaxBytesToEmit in the higher bits and references a
 199: // per-section marker symbol.
 200: bool LoongArchAsmBackend::relaxAlign(MCFragment &F, unsigned &Size) {
 201:   // Alignments before the first linker-relaxable instruction have fixed sizes
 202:   // and do not require relocations. Alignments after a linker-relaxable
 203:   // instruction require a relocation, even if the STI specifies norelax.
 204:   //
 205:   // firstLinkerRelaxable is the layout order within the subsection, which may
 206:   // be smaller than the section's order. Therefore, alignments in a
 207:   // lower-numbered subsection may be unnecessarily treated as linker-relaxable.
 208:   auto *Sec = F.getParent();
 209:   if (F.getLayoutOrder() <= Sec->firstLinkerRelaxable())
 210:     return false;
 211: 
 212:   // Use default handling unless linker relaxation is enabled and the
 213:   // MaxBytesToEmit >= the nop size.
 214:   const unsigned MinNopLen = 4;
 215:   unsigned MaxBytesToEmit = F.getAlignMaxBytesToEmit();
 216:   if (MaxBytesToEmit < MinNopLen)
 217:     return false;
 218: 
 219:   Size = F.getAlignment().value() - MinNopLen;
 220:   if (F.getAlignment() <= MinNopLen)
 221:     return false;
 222: 
 223:   MCContext &Ctx = getContext();
 224:   const MCExpr *Expr = nullptr;
```
- **EN**: The range implements or declares functions including `LoongArchAsmBackend::relaxAlign`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchAsmBackend::relaxAlign` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 225-252 / 第 225-252 行
```cpp
 225:   if (MaxBytesToEmit >= Size) {
 226:     Expr = MCConstantExpr::create(Size, getContext());
 227:   } else {
 228:     MCSection *Sec = F.getParent();
 229:     const MCSymbolRefExpr *SymRef = getSecToAlignSym()[Sec];
 230:     if (SymRef == nullptr) {
 231:       // Define a marker symbol at the section with an offset of 0.
 232:       MCSymbol *Sym = Ctx.createNamedTempSymbol("la-relax-align");
 233:       Sym->setFragment(&*Sec->getBeginSymbol()->getFragment());
 234:       Asm->registerSymbol(*Sym);
 235:       SymRef = MCSymbolRefExpr::create(Sym, Ctx);
 236:       getSecToAlignSym()[Sec] = SymRef;
 237:     }
 238:     Expr = MCBinaryExpr::createAdd(
 239:         SymRef,
 240:         MCConstantExpr::create((MaxBytesToEmit << 8) | Log2(F.getAlignment()),
 241:                                Ctx),
 242:         Ctx);
 243:   }
 244:   MCFixup Fixup =
 245:       MCFixup::create(0, Expr, FirstLiteralRelocationKind + ELF::R_LARCH_ALIGN);
 246:   F.setVarFixups({Fixup});
 247:   F.setLinkerRelaxable();
 248:   return true;
 249: }
 250: 
 251: std::pair<bool, bool> LoongArchAsmBackend::relaxLEB128(MCFragment &F,
 252:                                                        int64_t &Value) const {
```
- **EN**: The range implements or declares functions including `LoongArchAsmBackend::relaxLEB128`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchAsmBackend::relaxLEB128` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 253-280 / 第 253-280 行
```cpp
 253:   const MCExpr &Expr = F.getLEBValue();
 254:   if (F.isLEBSigned() || !Expr.evaluateKnownAbsolute(Value, *Asm))
 255:     return std::make_pair(false, false);
 256:   F.setVarFixups({MCFixup::create(0, &Expr, FK_Data_leb128)});
 257:   return std::make_pair(true, true);
 258: }
 259: 
 260: bool LoongArchAsmBackend::relaxDwarfLineAddr(MCFragment &F) const {
 261:   MCContext &C = getContext();
 262:   int64_t LineDelta = F.getDwarfLineDelta();
 263:   const MCExpr &AddrDelta = F.getDwarfAddrDelta();
 264:   int64_t Value;
 265:   if (AddrDelta.evaluateAsAbsolute(Value, *Asm))
 266:     return false;
 267:   [[maybe_unused]] bool IsAbsolute =
 268:       AddrDelta.evaluateKnownAbsolute(Value, *Asm);
 269:   assert(IsAbsolute);
 270: 
 271:   SmallVector<char> Data;
 272:   raw_svector_ostream OS(Data);
 273: 
 274:   // INT64_MAX is a signal that this is actually a DW_LNE_end_sequence.
 275:   if (LineDelta != INT64_MAX) {
 276:     OS << uint8_t(dwarf::DW_LNS_advance_line);
 277:     encodeSLEB128(LineDelta, OS);
 278:   }
 279: 
 280:   // According to the DWARF specification, the `DW_LNS_fixed_advance_pc` opcode
```
- **EN**: The range implements or declares functions including `LoongArchAsmBackend::relaxDwarfLineAddr`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchAsmBackend::relaxDwarfLineAddr` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 281-308 / 第 281-308 行
```cpp
 281:   // takes a single unsigned half (unencoded) operand. The maximum encodable
 282:   // value is therefore 65535.  Set a conservative upper bound for relaxation.
 283:   unsigned PCBytes;
 284:   if (Value > 60000) {
 285:     unsigned PtrSize = C.getAsmInfo().getCodePointerSize();
 286:     assert((PtrSize == 4 || PtrSize == 8) && "Unexpected pointer size");
 287:     PCBytes = PtrSize;
 288:     OS << uint8_t(dwarf::DW_LNS_extended_op) << uint8_t(PtrSize + 1)
 289:        << uint8_t(dwarf::DW_LNE_set_address);
 290:     OS.write_zeros(PtrSize);
 291:   } else {
 292:     PCBytes = 2;
 293:     OS << uint8_t(dwarf::DW_LNS_fixed_advance_pc);
 294:     support::endian::write<uint16_t>(OS, 0, llvm::endianness::little);
 295:   }
 296:   auto Offset = OS.tell() - PCBytes;
 297: 
 298:   if (LineDelta == INT64_MAX) {
 299:     OS << uint8_t(dwarf::DW_LNS_extended_op);
 300:     OS << uint8_t(1);
 301:     OS << uint8_t(dwarf::DW_LNE_end_sequence);
 302:   } else {
 303:     OS << uint8_t(dwarf::DW_LNS_copy);
 304:   }
 305: 
 306:   F.setVarContents(Data);
 307:   F.setVarFixups({MCFixup::create(Offset, &AddrDelta,
 308:                                   MCFixup::getDataKindForSize(PCBytes))});
```
- **EN**: The range implements or declares functions including `uint8_t`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `uint8_t` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 309-336 / 第 309-336 行
```cpp
 309:   return true;
 310: }
 311: 
 312: bool LoongArchAsmBackend::relaxDwarfCFA(MCFragment &F) const {
 313:   const MCExpr &AddrDelta = F.getDwarfAddrDelta();
 314:   SmallVector<MCFixup, 2> Fixups;
 315:   int64_t Value;
 316:   if (AddrDelta.evaluateAsAbsolute(Value, *Asm))
 317:     return false;
 318:   bool IsAbsolute = AddrDelta.evaluateKnownAbsolute(Value, *Asm);
 319:   assert(IsAbsolute && "CFA with invalid expression");
 320:   (void)IsAbsolute;
 321: 
 322:   assert(getContext().getAsmInfo().getMinInstAlignment() == 1 &&
 323:          "expected 1-byte alignment");
 324:   if (Value == 0) {
 325:     F.clearVarContents();
 326:     F.clearVarFixups();
 327:     return true;
 328:   }
 329: 
 330:   auto AddFixups = [&Fixups,
 331:                     &AddrDelta](unsigned Offset,
 332:                                 std::pair<MCFixupKind, MCFixupKind> FK) {
 333:     const MCBinaryExpr &MBE = cast<MCBinaryExpr>(AddrDelta);
 334:     Fixups.push_back(MCFixup::create(Offset, MBE.getLHS(), std::get<0>(FK)));
 335:     Fixups.push_back(MCFixup::create(Offset, MBE.getRHS(), std::get<1>(FK)));
 336:   };
```
- **EN**: The range implements or declares functions including `LoongArchAsmBackend::relaxDwarfCFA`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchAsmBackend::relaxDwarfCFA` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 337-364 / 第 337-364 行
```cpp
 337: 
 338:   SmallVector<char, 8> Data;
 339:   raw_svector_ostream OS(Data);
 340:   if (isUIntN(6, Value)) {
 341:     OS << uint8_t(dwarf::DW_CFA_advance_loc);
 342:     AddFixups(0, getRelocPairForSize(6));
 343:   } else if (isUInt<8>(Value)) {
 344:     OS << uint8_t(dwarf::DW_CFA_advance_loc1);
 345:     support::endian::write<uint8_t>(OS, 0, llvm::endianness::little);
 346:     AddFixups(1, getRelocPairForSize(8));
 347:   } else if (isUInt<16>(Value)) {
 348:     OS << uint8_t(dwarf::DW_CFA_advance_loc2);
 349:     support::endian::write<uint16_t>(OS, 0, llvm::endianness::little);
 350:     AddFixups(1, getRelocPairForSize(16));
 351:   } else if (isUInt<32>(Value)) {
 352:     OS << uint8_t(dwarf::DW_CFA_advance_loc4);
 353:     support::endian::write<uint32_t>(OS, 0, llvm::endianness::little);
 354:     AddFixups(1, getRelocPairForSize(32));
 355:   } else {
 356:     llvm_unreachable("unsupported CFA encoding");
 357:   }
 358:   F.setVarContents(Data);
 359:   F.setVarFixups(Fixups);
 360:   return true;
 361: }
 362: 
 363: bool LoongArchAsmBackend::writeNopData(raw_ostream &OS, uint64_t Count,
 364:                                        const MCSubtargetInfo *STI) const {
```
- **EN**: The range implements or declares functions including `LoongArchAsmBackend::writeNopData`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchAsmBackend::writeNopData` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 365-392 / 第 365-392 行
```cpp
 365:   // We mostly follow binutils' convention here: align to 4-byte boundary with a
 366:   // 0-fill padding.
 367:   OS.write_zeros(Count % 4);
 368: 
 369:   // The remainder is now padded with 4-byte nops.
 370:   // nop: andi r0, r0, 0
 371:   for (; Count >= 4; Count -= 4)
 372:     OS.write("\0\0\x40\x03", 4);
 373: 
 374:   return true;
 375: }
 376: 
 377: bool LoongArchAsmBackend::isPCRelFixupResolved(const MCSymbol *SymA,
 378:                                                const MCFragment &F) {
 379:   // If the section does not contain linker-relaxable fragments, PC-relative
 380:   // fixups can be resolved.
 381:   if (!F.getParent()->isLinkerRelaxable())
 382:     return true;
 383: 
 384:   // Otherwise, check if the offset between the symbol and fragment is fully
 385:   // resolved, unaffected by linker-relaxable fragments (e.g. instructions or
 386:   // offset-affected FT_Align fragments). Complements the generic
 387:   // isSymbolRefDifferenceFullyResolvedImpl.
 388:   if (!PCRelTemp)
 389:     PCRelTemp = getContext().createTempSymbol();
 390:   PCRelTemp->setFragment(const_cast<MCFragment *>(&F));
 391:   MCValue Res;
 392:   MCExpr::evaluateSymbolicAdd(Asm, false, MCValue::get(SymA),
```
- **EN**: The range implements or declares functions including `LoongArchAsmBackend::isPCRelFixupResolved`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchAsmBackend::isPCRelFixupResolved` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 393-420 / 第 393-420 行
```cpp
 393:                               MCValue::get(nullptr, PCRelTemp), Res);
 394:   return !Res.getSubSym();
 395: }
 396: 
 397: bool LoongArchAsmBackend::addReloc(const MCFragment &F, const MCFixup &Fixup,
 398:                                    const MCValue &Target, uint64_t &FixedValue,
 399:                                    bool IsResolved) {
 400:   auto Fallback = [&]() {
 401:     MCAsmBackend::maybeAddReloc(F, Fixup, Target, FixedValue, IsResolved);
 402:     return true;
 403:   };
 404:   uint64_t FixedValueA, FixedValueB;
 405:   if (Target.getSubSym()) {
 406:     // It's possible for Target to have (SymB != nullptr && SymA == nullptr).
 407:     // Go to the fallback path when we encounter this. See also #196927.
 408:     if (!Target.getAddSym())
 409:       return Fallback();
 410: 
 411:     assert(Target.getSpecifier() == 0 &&
 412:            "relocatable SymA-SymB cannot have relocation specifier");
 413:     std::pair<MCFixupKind, MCFixupKind> FK;
 414:     const MCSymbol &SA = *Target.getAddSym();
 415:     const MCSymbol &SB = *Target.getSubSym();
 416: 
 417:     bool force = !SA.isInSection() || !SB.isInSection();
 418:     if (!force) {
 419:       const MCSection &SecA = SA.getSection();
 420:       const MCSection &SecB = SB.getSection();
```
- **EN**: The range implements or declares functions including `LoongArchAsmBackend::addReloc`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchAsmBackend::addReloc` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 421-448 / 第 421-448 行
```cpp
 421:       const MCSection &SecCur = *F.getParent();
 422: 
 423:       // To handle the case of A - B which B is same section with the current,
 424:       // generate PCRel relocations is better than ADD/SUB relocation pair.
 425:       // We can resolve it as A - PC + PC - B. The A - PC will be resolved
 426:       // as a PCRel relocation, while PC - B will serve as the addend.
 427:       // If the linker relaxation is disabled, it can be done directly since
 428:       // PC - B is constant. Otherwise, we should evaluate whether PC - B
 429:       // is constant. If it can be resolved as PCRel, use Fallback which
 430:       // generates R_LARCH_{32,64}_PCREL relocation later.
 431:       if (&SecA != &SecB && &SecB == &SecCur &&
 432:           isPCRelFixupResolved(Target.getSubSym(), F))
 433:         return Fallback();
 434: 
 435:       if (&SecA == &SecB) {
 436:         // If the section is not linker-relaxable, or if the fixup is in a .dwo
 437:         // section (where relocations are forbidden), we must resolve the
 438:         // difference directly. The computed Value in evaluateFixup is correct
 439:         // based on the current layout.
 440:         if (!SecA.isLinkerRelaxable() || SecCur.getName().ends_with(".dwo"))
 441:           return true;
 442:       }
 443:     }
 444: 
 445:     switch (Fixup.getKind()) {
 446:     case llvm::FK_Data_1:
 447:       FK = getRelocPairForSize(8);
 448:       break;
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 449-476 / 第 449-476 行
```cpp
 449:     case llvm::FK_Data_2:
 450:       FK = getRelocPairForSize(16);
 451:       break;
 452:     case llvm::FK_Data_4:
 453:       FK = getRelocPairForSize(32);
 454:       break;
 455:     case llvm::FK_Data_8:
 456:       FK = getRelocPairForSize(64);
 457:       break;
 458:     case llvm::FK_Data_leb128:
 459:       FK = getRelocPairForSize(128);
 460:       break;
 461:     default:
 462:       llvm_unreachable("unsupported fixup size");
 463:     }
 464:     MCValue A = MCValue::get(Target.getAddSym(), nullptr, Target.getConstant());
 465:     MCValue B = MCValue::get(Target.getSubSym());
 466:     auto FA = MCFixup::create(Fixup.getOffset(), nullptr, std::get<0>(FK));
 467:     auto FB = MCFixup::create(Fixup.getOffset(), nullptr, std::get<1>(FK));
 468:     Asm->getWriter().recordRelocation(F, FA, A, FixedValueA);
 469:     Asm->getWriter().recordRelocation(F, FB, B, FixedValueB);
 470:     FixedValue = FixedValueA - FixedValueB;
 471:     return false;
 472:   }
 473: 
 474:   // If linker relaxation is enabled and supported by the current relocation,
 475:   // generate a relocation and then append a RELAX.
 476:   if (Fixup.isLinkerRelaxable())
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 477-504 / 第 477-504 行
```cpp
 477:     IsResolved = false;
 478:   if (IsResolved && Fixup.isPCRel())
 479:     IsResolved = isPCRelFixupResolved(Target.getAddSym(), F);
 480: 
 481:   if (!IsResolved)
 482:     Asm->getWriter().recordRelocation(F, Fixup, Target, FixedValue);
 483: 
 484:   if (Fixup.isLinkerRelaxable()) {
 485:     auto FA = MCFixup::create(Fixup.getOffset(), nullptr, ELF::R_LARCH_RELAX);
 486:     Asm->getWriter().recordRelocation(F, FA, MCValue::get(nullptr),
 487:                                       FixedValueA);
 488:   }
 489: 
 490:   return true;
 491: }
 492: 
 493: std::unique_ptr<MCObjectTargetWriter>
 494: LoongArchAsmBackend::createObjectTargetWriter() const {
 495:   return createLoongArchELFObjectWriter(OSABI, Is64Bit);
 496: }
 497: 
 498: MCAsmBackend *llvm::createLoongArchAsmBackend(const Target &T,
 499:                                               const MCSubtargetInfo &STI,
 500:                                               const MCRegisterInfo &MRI,
 501:                                               const MCTargetOptions &Options) {
 502:   const Triple &TT = STI.getTargetTriple();
 503:   uint8_t OSABI = MCELFObjectTargetWriter::getOSABI(TT.getOS());
 504:   return new LoongArchAsmBackend(STI, OSABI, TT.isArch64Bit(), Options);
```
- **EN**: The range implements or declares functions including `LoongArchAsmBackend::createObjectTargetWriter`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchAsmBackend::createObjectTargetWriter` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 505-505 / 第 505-505 行
```cpp
 505: }
```
- **EN**: This span continues the file's main responsibility: this file connects the backend to LLVM MC layer descriptors for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `LoongArchAsmBackend.h`
- `LoongArchFixupKinds.h`
- `llvm/BinaryFormat/ELF.h`
- `llvm/MC/MCAsmInfo.h`
- `llvm/MC/MCAssembler.h`
- `llvm/MC/MCContext.h`
- `llvm/MC/MCELFObjectWriter.h`
- `llvm/MC/MCExpr.h`
- `llvm/MC/MCSection.h`
- `llvm/MC/MCValue.h`
- `llvm/Support/EndianStream.h`
- `llvm/Support/LEB128.h`
- `llvm/Support/MathExtras.h`
- `llvm/BinaryFormat/ELFRelocs/LoongArch.def`
