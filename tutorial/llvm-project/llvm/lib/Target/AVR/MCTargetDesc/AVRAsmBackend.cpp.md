# AVRAsmBackend.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/AVR/MCTargetDesc/AVRAsmBackend.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file implements the AVRAsmBackend class.
- 目的（中文）: 实现 MC 汇编后端，包括重定位修正、指令松弛和目标文件输出策略。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- AVRAsmBackend.cpp - AVR Asm Backend  ------------------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file implements the AVRAsmBackend class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "MCTargetDesc/AVRAsmBackend.h"
  14: #include "MCTargetDesc/AVRFixupKinds.h"
  15: #include "MCTargetDesc/AVRMCTargetDesc.h"
  16: #include "llvm/ADT/StringSwitch.h"
  17: #include "llvm/MC/MCAsmBackend.h"
  18: #include "llvm/MC/MCAssembler.h"
  19: #include "llvm/MC/MCContext.h"
  20: #include "llvm/MC/MCELFObjectWriter.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

### Lines 21-40

```cpp
  21: #include "llvm/MC/MCExpr.h"
  22: #include "llvm/MC/MCObjectWriter.h"
  23: #include "llvm/MC/MCSubtargetInfo.h"
  24: #include "llvm/MC/MCValue.h"
  25: #include "llvm/Support/ErrorHandling.h"
  26: #include "llvm/Support/MathExtras.h"
  27: #include "llvm/Support/raw_ostream.h"
  28: 
  29: namespace adjust {
  30: 
  31: using namespace llvm;
  32: 
  33: static void unsigned_width(unsigned Width, uint64_t Value,
  34:                            std::string Description, const MCFixup &Fixup,
  35:                            MCContext *Ctx) {
  36:   if (!isUIntN(Width, Value)) {
  37:     std::string Diagnostic = "out of range " + Description;
  38: 
  39:     int64_t Max = maxUIntN(Width);
  40: 
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as unsigned_width contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 unsigned_width 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 41-60

```cpp
  41:     Diagnostic +=
  42:         " (expected an integer in the range 0 to " + std::to_string(Max) + ")";
  43: 
  44:     Ctx->reportError(Fixup.getLoc(), Diagnostic);
  45:   }
  46: }
  47: 
  48: /// Adjusts the value of a branch target before fixup application.
  49: static void adjustBranch(unsigned Size, const MCFixup &Fixup, uint64_t &Value,
  50:                          MCContext *Ctx) {
  51:   // We have one extra bit of precision because the value is rightshifted by
  52:   // one.
  53:   unsigned_width(Size + 1, Value, std::string("branch target"), Fixup, Ctx);
  54: 
  55:   // Rightshifts the value by one.
  56:   AVR::fixups::adjustBranchTarget(Value);
  57: }
  58: 
  59: /// Adjusts the value of a relative branch target before fixup application.
  60: static bool adjustRelativeBranch(unsigned Size, const MCFixup &Fixup,
```

- EN: Function bodies or method definitions such as adjustBranch contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: adjustBranch 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 61-80

```cpp
  61:                                  uint64_t &Value, const MCSubtargetInfo *STI) {
  62:   // Jumps are relative to the current instruction.
  63:   Value -= 2;
  64: 
  65:   // We have one extra bit of precision because the value is rightshifted by
  66:   // one.
  67:   Size += 1;
  68: 
  69:   assert(STI && "STI can not be NULL");
  70: 
  71:   if (!isIntN(Size, Value) && STI->hasFeature(AVR::FeatureWrappingRjmp)) {
  72:     const int32_t FlashSize = 0x2000;
  73:     int32_t SignedValue = Value;
  74: 
  75:     uint64_t WrappedValue = SignedValue > 0 ? (uint64_t)(Value - FlashSize)
  76:                                             : (uint64_t)(FlashSize + Value);
  77: 
  78:     if (isIntN(Size, WrappedValue)) {
  79:       Value = WrappedValue;
  80:     }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 81-100

```cpp
  81:   }
  82: 
  83:   if (!isIntN(Size, Value)) {
  84:     return false;
  85:   }
  86: 
  87:   // Rightshifts the value by one.
  88:   AVR::fixups::adjustBranchTarget(Value);
  89: 
  90:   return true;
  91: }
  92: 
  93: /// 22-bit absolute fixup.
  94: ///
  95: /// Resolves to:
  96: /// 1001 kkkk 010k kkkk kkkk kkkk 111k kkkk
  97: ///
  98: /// Offset of 0 (so the result is left shifted by 3 bits before application).
  99: static void fixup_call(unsigned Size, const MCFixup &Fixup, uint64_t &Value,
 100:                        MCContext *Ctx) {
```

- EN: Function bodies or method definitions such as fixup_call contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: fixup_call 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 101-120

```cpp
 101:   adjustBranch(Size, Fixup, Value, Ctx);
 102: 
 103:   auto top = Value & (0xf00000 << 6);   // the top four bits
 104:   auto middle = Value & (0x1ffff << 5); // the middle 13 bits
 105:   auto bottom = Value & 0x1f;           // end bottom 5 bits
 106: 
 107:   Value = (top << 6) | (middle << 3) | (bottom << 0);
 108: }
 109: 
 110: /// 7-bit PC-relative fixup.
 111: ///
 112: /// Resolves to:
 113: /// 0000 00kk kkkk k000
 114: /// Offset of 0 (so the result is left shifted by 3 bits before application).
 115: static void fixup_7_pcrel(unsigned Size, const MCFixup &Fixup, uint64_t &Value,
 116:                           MCContext *Ctx) {
 117:   if (!adjustRelativeBranch(Size, Fixup, Value, Ctx->getSubtargetInfo())) {
 118:     llvm_unreachable("should've been emitted as a relocation");
 119:   }
 120: 
```

- EN: Function bodies or method definitions such as fixup_7_pcrel contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: fixup_7_pcrel 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 121-140

```cpp
 121:   // Because the value may be negative, we must mask out the sign bits
 122:   Value &= 0x7f;
 123: }
 124: 
 125: /// 12-bit PC-relative fixup.
 126: /// Yes, the fixup is 12 bits even though the name says otherwise.
 127: ///
 128: /// Resolves to:
 129: /// 0000 kkkk kkkk kkkk
 130: /// Offset of 0 (so the result isn't left-shifted before application).
 131: static void fixup_13_pcrel(unsigned Size, const MCFixup &Fixup, uint64_t &Value,
 132:                            MCContext *Ctx) {
 133:   if (!adjustRelativeBranch(Size, Fixup, Value, Ctx->getSubtargetInfo())) {
 134:     llvm_unreachable("should've been emitted as a relocation");
 135:   }
 136: 
 137:   // Because the value may be negative, we must mask out the sign bits
 138:   Value &= 0xfff;
 139: }
 140: 
```

- EN: Function bodies or method definitions such as fixup_13_pcrel contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: fixup_13_pcrel 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 141-160

```cpp
 141: /// 6-bit fixup for the immediate operand of the STD/LDD family of
 142: /// instructions.
 143: ///
 144: /// Resolves to:
 145: /// 10q0 qq10 0000 1qqq
 146: static void fixup_6(const MCFixup &Fixup, uint64_t &Value, MCContext *Ctx) {
 147:   unsigned_width(6, Value, std::string("immediate"), Fixup, Ctx);
 148: 
 149:   Value = ((Value & 0x20) << 8) | ((Value & 0x18) << 7) | (Value & 0x07);
 150: }
 151: 
 152: /// 6-bit fixup for the immediate operand of the ADIW family of
 153: /// instructions.
 154: ///
 155: /// Resolves to:
 156: /// 0000 0000 kk00 kkkk
 157: static void fixup_6_adiw(const MCFixup &Fixup, uint64_t &Value,
 158:                          MCContext *Ctx) {
 159:   unsigned_width(6, Value, std::string("immediate"), Fixup, Ctx);
 160: 
```

- EN: Function bodies or method definitions such as fixup_6, fixup_6_adiw contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: fixup_6, fixup_6_adiw 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 161-180

```cpp
 161:   Value = ((Value & 0x30) << 2) | (Value & 0x0f);
 162: }
 163: 
 164: /// 5-bit port number fixup on the SBIC family of instructions.
 165: ///
 166: /// Resolves to:
 167: /// 0000 0000 AAAA A000
 168: static void fixup_port5(const MCFixup &Fixup, uint64_t &Value, MCContext *Ctx) {
 169:   unsigned_width(5, Value, std::string("port number"), Fixup, Ctx);
 170: 
 171:   Value &= 0x1f;
 172: 
 173:   Value <<= 3;
 174: }
 175: 
 176: /// 6-bit port number fixup on the IN family of instructions.
 177: ///
 178: /// Resolves to:
 179: /// 1011 0AAd dddd AAAA
 180: static void fixup_port6(const MCFixup &Fixup, uint64_t &Value, MCContext *Ctx) {
```

- EN: Function bodies or method definitions such as fixup_port5, fixup_port6 contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: fixup_port5, fixup_port6 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 181-200

```cpp
 181:   unsigned_width(6, Value, std::string("port number"), Fixup, Ctx);
 182: 
 183:   Value = ((Value & 0x30) << 5) | (Value & 0x0f);
 184: }
 185: 
 186: /// 7-bit data space address fixup for the LDS/STS instructions on AVRTiny.
 187: ///
 188: /// Resolves to:
 189: /// 1010 ikkk dddd kkkk
 190: static void fixup_lds_sts_16(const MCFixup &Fixup, uint64_t &Value,
 191:                              MCContext *Ctx) {
 192:   unsigned_width(7, Value, std::string("immediate"), Fixup, Ctx);
 193:   Value = ((Value & 0x70) << 8) | (Value & 0x0f);
 194: }
 195: 
 196: /// Adjusts a program memory address.
 197: /// This is a simple right-shift.
 198: static void pm(uint64_t &Value) { Value >>= 1; }
 199: 
 200: /// Fixups relating to the LDI instruction.
```

- EN: Function bodies or method definitions such as fixup_lds_sts_16, pm contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: fixup_lds_sts_16, pm 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 201-220

```cpp
 201: namespace ldi {
 202: 
 203: /// Adjusts a value to fix up the immediate of an `LDI Rd, K` instruction.
 204: ///
 205: /// Resolves to:
 206: /// 0000 KKKK 0000 KKKK
 207: /// Offset of 0 (so the result isn't left-shifted before application).
 208: static void fixup(unsigned Size, const MCFixup &Fixup, uint64_t &Value,
 209:                   MCContext *Ctx) {
 210:   uint64_t upper = Value & 0xf0;
 211:   uint64_t lower = Value & 0x0f;
 212: 
 213:   Value = (upper << 4) | lower;
 214: }
 215: 
 216: static void neg(uint64_t &Value) { Value *= -1; }
 217: 
 218: static void lo8(unsigned Size, const MCFixup &Fixup, uint64_t &Value,
 219:                 MCContext *Ctx) {
 220:   Value &= 0xff;
```

- EN: Function bodies or method definitions such as fixup, neg, lo8 contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: fixup, neg, lo8 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 221-240

```cpp
 221:   ldi::fixup(Size, Fixup, Value, Ctx);
 222: }
 223: 
 224: static void hi8(unsigned Size, const MCFixup &Fixup, uint64_t &Value,
 225:                 MCContext *Ctx) {
 226:   Value = (Value & 0xff00) >> 8;
 227:   ldi::fixup(Size, Fixup, Value, Ctx);
 228: }
 229: 
 230: static void hh8(unsigned Size, const MCFixup &Fixup, uint64_t &Value,
 231:                 MCContext *Ctx) {
 232:   Value = (Value & 0xff0000) >> 16;
 233:   ldi::fixup(Size, Fixup, Value, Ctx);
 234: }
 235: 
 236: static void ms8(unsigned Size, const MCFixup &Fixup, uint64_t &Value,
 237:                 MCContext *Ctx) {
 238:   Value = (Value & 0xff000000) >> 24;
 239:   ldi::fixup(Size, Fixup, Value, Ctx);
 240: }
```

- EN: Function bodies or method definitions such as hi8, hh8, ms8 contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: hi8, hh8, ms8 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 241-260

```cpp
 241: 
 242: } // namespace ldi
 243: } // namespace adjust
 244: 
 245: namespace llvm {
 246: 
 247: // Prepare value for the target space for it
 248: void AVRAsmBackend::adjustFixupValue(const MCFixup &Fixup,
 249:                                      const MCValue &Target, uint64_t &Value,
 250:                                      MCContext *Ctx) const {
 251:   // The size of the fixup in bits.
 252:   uint64_t Size = AVRAsmBackend::getFixupKindInfo(Fixup.getKind()).TargetSize;
 253: 
 254:   unsigned Kind = Fixup.getKind();
 255:   switch (Kind) {
 256:   default:
 257:     llvm_unreachable("unhandled fixup");
 258:   case AVR::fixup_7_pcrel:
 259:     adjust::fixup_7_pcrel(Size, Fixup, Value, Ctx);
 260:     break;
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as adjustFixupValue contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 adjustFixupValue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 261-280

```cpp
 261:   case AVR::fixup_13_pcrel:
 262:     adjust::fixup_13_pcrel(Size, Fixup, Value, Ctx);
 263:     break;
 264:   case AVR::fixup_call:
 265:     adjust::fixup_call(Size, Fixup, Value, Ctx);
 266:     break;
 267:   case AVR::fixup_ldi:
 268:     adjust::ldi::fixup(Size, Fixup, Value, Ctx);
 269:     break;
 270:   case AVR::fixup_lo8_ldi:
 271:     adjust::ldi::lo8(Size, Fixup, Value, Ctx);
 272:     break;
 273:   case AVR::fixup_lo8_ldi_pm:
 274:   case AVR::fixup_lo8_ldi_gs:
 275:     adjust::pm(Value);
 276:     adjust::ldi::lo8(Size, Fixup, Value, Ctx);
 277:     break;
 278:   case AVR::fixup_hi8_ldi:
 279:     adjust::ldi::hi8(Size, Fixup, Value, Ctx);
 280:     break;
```

- EN: This range continues the implementation of the backend component described by AVRAsmBackend.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 281-300

```cpp
 281:   case AVR::fixup_hi8_ldi_pm:
 282:   case AVR::fixup_hi8_ldi_gs:
 283:     adjust::pm(Value);
 284:     adjust::ldi::hi8(Size, Fixup, Value, Ctx);
 285:     break;
 286:   case AVR::fixup_hh8_ldi:
 287:   case AVR::fixup_hh8_ldi_pm:
 288:     if (Kind == AVR::fixup_hh8_ldi_pm)
 289:       adjust::pm(Value);
 290: 
 291:     adjust::ldi::hh8(Size, Fixup, Value, Ctx);
 292:     break;
 293:   case AVR::fixup_ms8_ldi:
 294:     adjust::ldi::ms8(Size, Fixup, Value, Ctx);
 295:     break;
 296: 
 297:   case AVR::fixup_lo8_ldi_neg:
 298:   case AVR::fixup_lo8_ldi_pm_neg:
 299:     if (Kind == AVR::fixup_lo8_ldi_pm_neg)
 300:       adjust::pm(Value);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 301-320

```cpp
 301: 
 302:     adjust::ldi::neg(Value);
 303:     adjust::ldi::lo8(Size, Fixup, Value, Ctx);
 304:     break;
 305:   case AVR::fixup_hi8_ldi_neg:
 306:   case AVR::fixup_hi8_ldi_pm_neg:
 307:     if (Kind == AVR::fixup_hi8_ldi_pm_neg)
 308:       adjust::pm(Value);
 309: 
 310:     adjust::ldi::neg(Value);
 311:     adjust::ldi::hi8(Size, Fixup, Value, Ctx);
 312:     break;
 313:   case AVR::fixup_hh8_ldi_neg:
 314:   case AVR::fixup_hh8_ldi_pm_neg:
 315:     if (Kind == AVR::fixup_hh8_ldi_pm_neg)
 316:       adjust::pm(Value);
 317: 
 318:     adjust::ldi::neg(Value);
 319:     adjust::ldi::hh8(Size, Fixup, Value, Ctx);
 320:     break;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 321-340

```cpp
 321:   case AVR::fixup_ms8_ldi_neg:
 322:     adjust::ldi::neg(Value);
 323:     adjust::ldi::ms8(Size, Fixup, Value, Ctx);
 324:     break;
 325:   case AVR::fixup_16:
 326:     adjust::unsigned_width(16, Value, std::string("port number"), Fixup, Ctx);
 327: 
 328:     Value &= 0xffff;
 329:     break;
 330:   case AVR::fixup_16_pm:
 331:     Value >>= 1; // Flash addresses are always shifted.
 332:     adjust::unsigned_width(16, Value, std::string("port number"), Fixup, Ctx);
 333: 
 334:     Value &= 0xffff;
 335:     break;
 336: 
 337:   case AVR::fixup_6:
 338:     adjust::fixup_6(Fixup, Value, Ctx);
 339:     break;
 340:   case AVR::fixup_6_adiw:
```

- EN: This range continues the implementation of the backend component described by AVRAsmBackend.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 341-360

```cpp
 341:     adjust::fixup_6_adiw(Fixup, Value, Ctx);
 342:     break;
 343: 
 344:   case AVR::fixup_port5:
 345:     adjust::fixup_port5(Fixup, Value, Ctx);
 346:     break;
 347: 
 348:   case AVR::fixup_port6:
 349:     adjust::fixup_port6(Fixup, Value, Ctx);
 350:     break;
 351: 
 352:   case AVR::fixup_lds_sts_16:
 353:     adjust::fixup_lds_sts_16(Fixup, Value, Ctx);
 354:     break;
 355: 
 356:   // Fixups which do not require adjustments.
 357:   case FK_Data_1:
 358:   case FK_Data_2:
 359:   case FK_Data_4:
 360:   case FK_Data_8:
```

- EN: This range continues the implementation of the backend component described by AVRAsmBackend.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 361-380

```cpp
 361:     break;
 362:   }
 363: }
 364: 
 365: std::unique_ptr<MCObjectTargetWriter>
 366: AVRAsmBackend::createObjectTargetWriter() const {
 367:   return createAVRELFObjectWriter(MCELFObjectTargetWriter::getOSABI(OSType));
 368: }
 369: 
 370: void AVRAsmBackend::applyFixup(const MCFragment &F, const MCFixup &Fixup,
 371:                                const MCValue &Target, uint8_t *Data,
 372:                                uint64_t Value, bool IsResolved) {
 373:   // AVR sets the fixup value to bypass the assembly time overflow with a
 374:   // relocation.
 375:   if (IsResolved) {
 376:     auto TargetVal = MCValue::get(Target.getAddSym(), Target.getSubSym(), Value,
 377:                                   Target.getSpecifier());
 378:     if (forceRelocation(F, Fixup, TargetVal))
 379:       IsResolved = false;
 380:   }
```

- EN: Function bodies or method definitions such as createObjectTargetWriter, applyFixup contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: createObjectTargetWriter, applyFixup 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 381-400

```cpp
 381:   if (!IsResolved)
 382:     Asm->getWriter().recordRelocation(F, Fixup, Target, Value);
 383: 
 384:   if (mc::isRelocation(Fixup.getKind()))
 385:     return;
 386:   adjustFixupValue(Fixup, Target, Value, &getContext());
 387:   if (Value == 0)
 388:     return; // Doesn't change encoding.
 389: 
 390:   MCFixupKindInfo Info = getFixupKindInfo(Fixup.getKind());
 391: 
 392:   // The number of bits in the fixup mask
 393:   unsigned NumBits = Info.TargetSize + Info.TargetOffset;
 394:   auto NumBytes = (NumBits / 8) + ((NumBits % 8) == 0 ? 0 : 1);
 395: 
 396:   // Shift the value into position.
 397:   Value <<= Info.TargetOffset;
 398: 
 399:   assert(Fixup.getOffset() + NumBytes <= F.getSize() &&
 400:          "Invalid fixup offset!");
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 401-420

```cpp
 401: 
 402:   // For each byte of the fragment that the fixup touches, mask in the
 403:   // bits from the fixup value.
 404:   for (unsigned i = 0; i < NumBytes; ++i) {
 405:     uint8_t mask = (((Value >> (i * 8)) & 0xff));
 406:     Data[i] |= mask;
 407:   }
 408: }
 409: 
 410: std::optional<MCFixupKind> AVRAsmBackend::getFixupKind(StringRef Name) const {
 411:   unsigned Type;
 412:   Type = llvm::StringSwitch<unsigned>(Name)
 413: #define ELF_RELOC(X, Y) .Case(#X, Y)
 414: #include "llvm/BinaryFormat/ELFRelocs/AVR.def"
 415: #undef ELF_RELOC
 416:              .Case("BFD_RELOC_NONE", ELF::R_AVR_NONE)
 417:              .Case("BFD_RELOC_16", ELF::R_AVR_16)
 418:              .Case("BFD_RELOC_32", ELF::R_AVR_32)
 419:              .Default(-1u);
 420:   if (Type != -1u)
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. Function bodies or method definitions such as getFixupKind contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 getFixupKind 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 421-440

```cpp
 421:     return static_cast<MCFixupKind>(FirstLiteralRelocationKind + Type);
 422:   return std::nullopt;
 423: }
 424: 
 425: MCFixupKindInfo AVRAsmBackend::getFixupKindInfo(MCFixupKind Kind) const {
 426:   // NOTE: Many AVR fixups work on sets of non-contignous bits. We work around
 427:   // this by saying that the fixup is the size of the entire instruction.
 428:   const static MCFixupKindInfo Infos[AVR::NumTargetFixupKinds] = {
 429:       // This table *must* be in same the order of fixup_* kinds in
 430:       // AVRFixupKinds.h.
 431:       //
 432:       // name                    offset  bits  flags
 433:       {"fixup_32", 0, 32, 0},
 434: 
 435:       {"fixup_7_pcrel", 3, 7, 0},
 436:       {"fixup_13_pcrel", 0, 12, 0},
 437: 
 438:       {"fixup_16", 0, 16, 0},
 439:       {"fixup_16_pm", 0, 16, 0},
 440: 
```

- EN: Function bodies or method definitions such as getFixupKindInfo contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: getFixupKindInfo 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 441-460

```cpp
 441:       {"fixup_ldi", 0, 8, 0},
 442: 
 443:       {"fixup_lo8_ldi", 0, 8, 0},
 444:       {"fixup_hi8_ldi", 0, 8, 0},
 445:       {"fixup_hh8_ldi", 0, 8, 0},
 446:       {"fixup_ms8_ldi", 0, 8, 0},
 447: 
 448:       {"fixup_lo8_ldi_neg", 0, 8, 0},
 449:       {"fixup_hi8_ldi_neg", 0, 8, 0},
 450:       {"fixup_hh8_ldi_neg", 0, 8, 0},
 451:       {"fixup_ms8_ldi_neg", 0, 8, 0},
 452: 
 453:       {"fixup_lo8_ldi_pm", 0, 8, 0},
 454:       {"fixup_hi8_ldi_pm", 0, 8, 0},
 455:       {"fixup_hh8_ldi_pm", 0, 8, 0},
 456: 
 457:       {"fixup_lo8_ldi_pm_neg", 0, 8, 0},
 458:       {"fixup_hi8_ldi_pm_neg", 0, 8, 0},
 459:       {"fixup_hh8_ldi_pm_neg", 0, 8, 0},
 460: 
```

- EN: This range continues the implementation of the backend component described by AVRAsmBackend.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 461-480

```cpp
 461:       {"fixup_call", 0, 22, 0},
 462: 
 463:       {"fixup_6", 0, 16, 0}, // non-contiguous
 464:       {"fixup_6_adiw", 0, 6, 0},
 465: 
 466:       {"fixup_lo8_ldi_gs", 0, 8, 0},
 467:       {"fixup_hi8_ldi_gs", 0, 8, 0},
 468: 
 469:       {"fixup_8", 0, 8, 0},
 470:       {"fixup_8_lo8", 0, 8, 0},
 471:       {"fixup_8_hi8", 0, 8, 0},
 472:       {"fixup_8_hlo8", 0, 8, 0},
 473: 
 474:       {"fixup_diff8", 0, 8, 0},
 475:       {"fixup_diff16", 0, 16, 0},
 476:       {"fixup_diff32", 0, 32, 0},
 477: 
 478:       {"fixup_lds_sts_16", 0, 16, 0},
 479: 
 480:       {"fixup_port6", 0, 16, 0}, // non-contiguous
```

- EN: This range continues the implementation of the backend component described by AVRAsmBackend.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 481-500

```cpp
 481:       {"fixup_port5", 3, 5, 0},
 482:   };
 483: 
 484:   // Fixup kinds from .reloc directive are like R_AVR_NONE. They do not require
 485:   // any extra processing.
 486:   if (mc::isRelocation(Kind))
 487:     return {};
 488: 
 489:   if (Kind < FirstTargetFixupKind)
 490:     return MCAsmBackend::getFixupKindInfo(Kind);
 491: 
 492:   assert(unsigned(Kind - FirstTargetFixupKind) < AVR::NumTargetFixupKinds &&
 493:          "Invalid kind!");
 494: 
 495:   return Infos[Kind - FirstTargetFixupKind];
 496: }
 497: 
 498: bool AVRAsmBackend::writeNopData(raw_ostream &OS, uint64_t Count,
 499:                                  const MCSubtargetInfo *STI) const {
 500:   // If the count is not 2-byte aligned, we must be writing data into the text
```

- EN: Function bodies or method definitions such as writeNopData contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: writeNopData 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 501-520

```cpp
 501:   // section (otherwise we have unaligned instructions, and thus have far
 502:   // bigger problems), so just write zeros instead.
 503:   assert((Count % 2) == 0 && "NOP instructions must be 2 bytes");
 504: 
 505:   OS.write_zeros(Count);
 506:   return true;
 507: }
 508: 
 509: bool AVRAsmBackend::forceRelocation(const MCFragment &F, const MCFixup &Fixup,
 510:                                     const MCValue &Target) {
 511:   switch ((unsigned)Fixup.getKind()) {
 512:   default:
 513:     return false;
 514: 
 515:   case AVR::fixup_7_pcrel:
 516:   case AVR::fixup_13_pcrel:
 517:   case AVR::fixup_call:
 518:     return true;
 519:   }
 520: }
```

- EN: Function bodies or method definitions such as forceRelocation contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: forceRelocation 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 521-528

```cpp
 521: 
 522: MCAsmBackend *createAVRAsmBackend(const Target &T, const MCSubtargetInfo &STI,
 523:                                   const MCRegisterInfo &MRI,
 524:                                   const llvm::MCTargetOptions &TO) {
 525:   return new AVRAsmBackend(STI.getTargetTriple().getOS());
 526: }
 527: 
 528: } // end of namespace llvm
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。

## Key Concepts / 关键概念

- Fixups and relaxation / 修正与松弛
- Object emission rules / 目标文件输出规则
- Assembler backend policies / 汇编后端策略
- CPU feature modelling / CPU 特性建模
- ELF object support / ELF 目标文件支持

## Dependencies / 依赖关系

- Direct includes / 直接包含: `MCTargetDesc/AVRAsmBackend.h`, `MCTargetDesc/AVRFixupKinds.h`, `MCTargetDesc/AVRMCTargetDesc.h`, `llvm/ADT/StringSwitch.h`, `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCAssembler.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCELFObjectWriter.h`
- LLVM subsystems / LLVM 子系统: LLVM MC
- Local companions / 本地配套文件: `AVRAsmBackend.h`
