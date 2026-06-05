# CSKYELFStreamer.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/MCTargetDesc/CSKYELFStreamer.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file provides CSKY specific target streamer methods.
- 目的（中文）: 提供目标后端使用的专用支持代码、声明或辅助例程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- CSKYELFStreamer.cpp - CSKY ELF Target Streamer Methods ------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file provides CSKY specific target streamer methods.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "CSKYELFStreamer.h"
  14: #include "CSKYMCTargetDesc.h"
  15: #include "MCTargetDesc/CSKYAsmBackend.h"
  16: #include "MCTargetDesc/CSKYBaseInfo.h"
  17: #include "llvm/BinaryFormat/ELF.h"
  18: #include "llvm/MC/MCAssembler.h"
  19: #include "llvm/MC/MCContext.h"
  20: #include "llvm/MC/MCELFObjectWriter.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

### Lines 21-40

```cpp
  21: #include "llvm/MC/MCSectionELF.h"
  22: #include "llvm/MC/MCSubtargetInfo.h"
  23: #include "llvm/MC/MCSymbolELF.h"
  24: #include "llvm/Support/CSKYAttributes.h"
  25: #include "llvm/Support/Casting.h"
  26: #include "llvm/Support/LEB128.h"
  27: #include "llvm/TargetParser/CSKYTargetParser.h"
  28: 
  29: using namespace llvm;
  30: 
  31: // This part is for ELF object output.
  32: CSKYTargetELFStreamer::CSKYTargetELFStreamer(MCStreamer &S,
  33:                                              const MCSubtargetInfo &STI)
  34:     : CSKYTargetStreamer(S), CurrentVendor("csky") {
  35:   ELFObjectWriter &W = getStreamer().getWriter();
  36:   const FeatureBitset &Features = STI.getFeatureBits();
  37: 
  38:   unsigned EFlags = W.getELFHeaderEFlags();
  39: 
  40:   EFlags |= ELF::EF_CSKY_ABIV2;
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as CSKYTargetStreamer contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 CSKYTargetStreamer 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 41-60

```cpp
  41: 
  42:   if (Features[CSKY::ProcCK801])
  43:     EFlags |= ELF::EF_CSKY_801;
  44:   else if (Features[CSKY::ProcCK802])
  45:     EFlags |= ELF::EF_CSKY_802;
  46:   else if (Features[CSKY::ProcCK803])
  47:     EFlags |= ELF::EF_CSKY_803;
  48:   else if (Features[CSKY::ProcCK804])
  49:     EFlags |= ELF::EF_CSKY_803;
  50:   else if (Features[CSKY::ProcCK805])
  51:     EFlags |= ELF::EF_CSKY_805;
  52:   else if (Features[CSKY::ProcCK807])
  53:     EFlags |= ELF::EF_CSKY_807;
  54:   else if (Features[CSKY::ProcCK810])
  55:     EFlags |= ELF::EF_CSKY_810;
  56:   else if (Features[CSKY::ProcCK860])
  57:     EFlags |= ELF::EF_CSKY_860;
  58:   else
  59:     EFlags |= ELF::EF_CSKY_810;
  60: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 61-80

```cpp
  61:   if (Features[CSKY::FeatureFPUV2_SF] || Features[CSKY::FeatureFPUV3_SF])
  62:     EFlags |= ELF::EF_CSKY_FLOAT;
  63: 
  64:   EFlags |= ELF::EF_CSKY_EFV1;
  65: 
  66:   W.setELFHeaderEFlags(EFlags);
  67: }
  68: 
  69: MCELFStreamer &CSKYTargetELFStreamer::getStreamer() {
  70:   return static_cast<MCELFStreamer &>(Streamer);
  71: }
  72: 
  73: void CSKYTargetELFStreamer::emitAttribute(unsigned Attribute, unsigned Value) {
  74:   setAttributeItem(Attribute, Value, /*OverwriteExisting=*/true);
  75: }
  76: 
  77: void CSKYTargetELFStreamer::emitTextAttribute(unsigned Attribute,
  78:                                               StringRef String) {
  79:   setAttributeItem(Attribute, String, /*OverwriteExisting=*/true);
  80: }
```

- EN: Function bodies or method definitions such as emitAttribute, emitTextAttribute contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: emitAttribute, emitTextAttribute 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 81-100

```cpp
  81: 
  82: void CSKYTargetELFStreamer::finishAttributeSection() {
  83:   if (Contents.empty())
  84:     return;
  85: 
  86:   if (AttributeSection) {
  87:     Streamer.switchSection(AttributeSection);
  88:   } else {
  89:     MCAssembler &MCA = getStreamer().getAssembler();
  90:     AttributeSection = MCA.getContext().getELFSection(
  91:         ".csky.attributes", ELF::SHT_CSKY_ATTRIBUTES, 0);
  92:     Streamer.switchSection(AttributeSection);
  93:     Streamer.emitInt8(ELFAttrs::Format_Version);
  94:   }
  95: 
  96:   // Vendor size + Vendor name + '\0'
  97:   const size_t VendorHeaderSize = 4 + CurrentVendor.size() + 1;
  98: 
  99:   // Tag + Tag Size
 100:   const size_t TagHeaderSize = 1 + 4;
```

- EN: Function bodies or method definitions such as finishAttributeSection contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: finishAttributeSection 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 101-120

```cpp
 101: 
 102:   const size_t ContentsSize = calculateContentSize();
 103: 
 104:   Streamer.emitInt32(VendorHeaderSize + TagHeaderSize + ContentsSize);
 105:   Streamer.emitBytes(CurrentVendor);
 106:   Streamer.emitInt8(0); // '\0'
 107: 
 108:   Streamer.emitInt8(ELFAttrs::File);
 109:   Streamer.emitInt32(TagHeaderSize + ContentsSize);
 110: 
 111:   // Size should have been accounted for already, now
 112:   // emit each field as its type (ULEB or String).
 113:   for (AttributeItem item : Contents) {
 114:     Streamer.emitULEB128IntValue(item.Tag);
 115:     switch (item.Type) {
 116:     default:
 117:       llvm_unreachable("Invalid attribute type");
 118:     case AttributeType::Numeric:
 119:       Streamer.emitULEB128IntValue(item.IntValue);
 120:       break;
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

### Lines 121-140

```cpp
 121:     case AttributeType::Text:
 122:       Streamer.emitBytes(item.StringValue);
 123:       Streamer.emitInt8(0); // '\0'
 124:       break;
 125:     case AttributeType::NumericAndText:
 126:       Streamer.emitULEB128IntValue(item.IntValue);
 127:       Streamer.emitBytes(item.StringValue);
 128:       Streamer.emitInt8(0); // '\0'
 129:       break;
 130:     }
 131:   }
 132: 
 133:   Contents.clear();
 134: }
 135: 
 136: size_t CSKYTargetELFStreamer::calculateContentSize() const {
 137:   size_t Result = 0;
 138:   for (AttributeItem item : Contents) {
 139:     switch (item.Type) {
 140:     case AttributeType::Hidden:
```

- EN: Function bodies or method definitions such as calculateContentSize contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: calculateContentSize 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

### Lines 141-160

```cpp
 141:       break;
 142:     case AttributeType::Numeric:
 143:       Result += getULEB128Size(item.Tag);
 144:       Result += getULEB128Size(item.IntValue);
 145:       break;
 146:     case AttributeType::Text:
 147:       Result += getULEB128Size(item.Tag);
 148:       Result += item.StringValue.size() + 1; // string + '\0'
 149:       break;
 150:     case AttributeType::NumericAndText:
 151:       Result += getULEB128Size(item.Tag);
 152:       Result += getULEB128Size(item.IntValue);
 153:       Result += item.StringValue.size() + 1; // string + '\0';
 154:       break;
 155:     }
 156:   }
 157:   return Result;
 158: }
 159: 
 160: void CSKYELFStreamer::EmitMappingSymbol(StringRef Name) {
```

- EN: Function bodies or method definitions such as EmitMappingSymbol contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: EmitMappingSymbol 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 161-180

```cpp
 161:   if (Name == "$d" && State == EMS_Data)
 162:     return;
 163:   if (Name == "$t" && State == EMS_Text)
 164:     return;
 165:   if (Name == "$t" && State == EMS_None) {
 166:     State = EMS_Text;
 167:     return;
 168:   }
 169: 
 170:   State = (Name == "$t" ? EMS_Text : EMS_Data);
 171: 
 172:   auto *Symbol =
 173:       static_cast<MCSymbolELF *>(getContext().createLocalSymbol(Name));
 174:   emitLabel(Symbol);
 175: 
 176:   Symbol->setType(ELF::STT_NOTYPE);
 177:   Symbol->setBinding(ELF::STB_LOCAL);
 178: }
 179: 
 180: void CSKYTargetELFStreamer::emitTargetAttributes(const MCSubtargetInfo &STI) {
```

- EN: Function bodies or method definitions such as emitTargetAttributes contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: emitTargetAttributes 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 181-200

```cpp
 181:   StringRef CPU = STI.getCPU();
 182:   CSKY::ArchKind ArchID = CSKY::parseCPUArch(CPU);
 183: 
 184:   if (ArchID == CSKY::ArchKind::CK804)
 185:     ArchID = CSKY::ArchKind::CK803;
 186: 
 187:   StringRef CPU_ARCH = CSKY::getArchName(ArchID);
 188: 
 189:   if (ArchID == CSKY::ArchKind::INVALID) {
 190:     CPU = "ck810";
 191:     CPU_ARCH = "ck810";
 192:   }
 193:   emitTextAttribute(CSKYAttrs::CSKY_ARCH_NAME, CPU_ARCH);
 194:   emitTextAttribute(CSKYAttrs::CSKY_CPU_NAME, CPU);
 195: 
 196:   unsigned ISAFlag = 0;
 197:   if (STI.hasFeature(CSKY::HasE1))
 198:     ISAFlag |= CSKYAttrs::V2_ISA_E1;
 199: 
 200:   if (STI.hasFeature(CSKY::HasE2))
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 201-220

```cpp
 201:     ISAFlag |= CSKYAttrs::V2_ISA_1E2;
 202: 
 203:   if (STI.hasFeature(CSKY::Has2E3))
 204:     ISAFlag |= CSKYAttrs::V2_ISA_2E3;
 205: 
 206:   if (STI.hasFeature(CSKY::HasMP))
 207:     ISAFlag |= CSKYAttrs::ISA_MP;
 208: 
 209:   if (STI.hasFeature(CSKY::Has3E3r1))
 210:     ISAFlag |= CSKYAttrs::V2_ISA_3E3R1;
 211: 
 212:   if (STI.hasFeature(CSKY::Has3r1E3r2))
 213:     ISAFlag |= CSKYAttrs::V2_ISA_3E3R2;
 214: 
 215:   if (STI.hasFeature(CSKY::Has3r2E3r3))
 216:     ISAFlag |= CSKYAttrs::V2_ISA_3E3R3;
 217: 
 218:   if (STI.hasFeature(CSKY::Has3E7))
 219:     ISAFlag |= CSKYAttrs::V2_ISA_3E7;
 220: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 221-240

```cpp
 221:   if (STI.hasFeature(CSKY::HasMP1E2))
 222:     ISAFlag |= CSKYAttrs::ISA_MP_1E2;
 223: 
 224:   if (STI.hasFeature(CSKY::Has7E10))
 225:     ISAFlag |= CSKYAttrs::V2_ISA_7E10;
 226: 
 227:   if (STI.hasFeature(CSKY::Has10E60))
 228:     ISAFlag |= CSKYAttrs::V2_ISA_10E60;
 229: 
 230:   if (STI.hasFeature(CSKY::FeatureTrust))
 231:     ISAFlag |= CSKYAttrs::ISA_TRUST;
 232: 
 233:   if (STI.hasFeature(CSKY::FeatureJAVA))
 234:     ISAFlag |= CSKYAttrs::ISA_JAVA;
 235: 
 236:   if (STI.hasFeature(CSKY::FeatureCache))
 237:     ISAFlag |= CSKYAttrs::ISA_CACHE;
 238: 
 239:   if (STI.hasFeature(CSKY::FeatureNVIC))
 240:     ISAFlag |= CSKYAttrs::ISA_NVIC;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 241-260

```cpp
 241: 
 242:   if (STI.hasFeature(CSKY::FeatureDSP))
 243:     ISAFlag |= CSKYAttrs::ISA_DSP;
 244: 
 245:   if (STI.hasFeature(CSKY::HasDSP1E2))
 246:     ISAFlag |= CSKYAttrs::ISA_DSP_1E2;
 247: 
 248:   if (STI.hasFeature(CSKY::HasDSPE60))
 249:     ISAFlag |= CSKYAttrs::V2_ISA_DSPE60;
 250: 
 251:   if (STI.hasFeature(CSKY::FeatureDSPV2))
 252:     ISAFlag |= CSKYAttrs::ISA_DSP_ENHANCE;
 253: 
 254:   if (STI.hasFeature(CSKY::FeatureDSP_Silan))
 255:     ISAFlag |= CSKYAttrs::ISA_DSP_SILAN;
 256: 
 257:   if (STI.hasFeature(CSKY::FeatureVDSPV1_128))
 258:     ISAFlag |= CSKYAttrs::ISA_VDSP;
 259: 
 260:   if (STI.hasFeature(CSKY::FeatureVDSPV2))
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 261-280

```cpp
 261:     ISAFlag |= CSKYAttrs::ISA_VDSP_2;
 262: 
 263:   if (STI.hasFeature(CSKY::HasVDSP2E3))
 264:     ISAFlag |= CSKYAttrs::ISA_VDSP_2E3;
 265: 
 266:   if (STI.hasFeature(CSKY::HasVDSP2E60F))
 267:     ISAFlag |= CSKYAttrs::ISA_VDSP_2E60F;
 268: 
 269:   emitAttribute(CSKYAttrs::CSKY_ISA_FLAGS, ISAFlag);
 270: 
 271:   unsigned ISAExtFlag = 0;
 272:   if (STI.hasFeature(CSKY::HasFLOATE1))
 273:     ISAExtFlag |= CSKYAttrs::ISA_FLOAT_E1;
 274: 
 275:   if (STI.hasFeature(CSKY::HasFLOAT1E2))
 276:     ISAExtFlag |= CSKYAttrs::ISA_FLOAT_1E2;
 277: 
 278:   if (STI.hasFeature(CSKY::HasFLOAT1E3))
 279:     ISAExtFlag |= CSKYAttrs::ISA_FLOAT_1E3;
 280: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 281-300

```cpp
 281:   if (STI.hasFeature(CSKY::HasFLOAT3E4))
 282:     ISAExtFlag |= CSKYAttrs::ISA_FLOAT_3E4;
 283: 
 284:   if (STI.hasFeature(CSKY::HasFLOAT7E60))
 285:     ISAExtFlag |= CSKYAttrs::ISA_FLOAT_7E60;
 286: 
 287:   emitAttribute(CSKYAttrs::CSKY_ISA_EXT_FLAGS, ISAExtFlag);
 288: 
 289:   if (STI.hasFeature(CSKY::FeatureDSP))
 290:     emitAttribute(CSKYAttrs::CSKY_DSP_VERSION,
 291:                   CSKYAttrs::DSP_VERSION_EXTENSION);
 292:   if (STI.hasFeature(CSKY::FeatureDSPV2))
 293:     emitAttribute(CSKYAttrs::CSKY_DSP_VERSION, CSKYAttrs::DSP_VERSION_2);
 294: 
 295:   if (STI.hasFeature(CSKY::FeatureVDSPV2))
 296:     emitAttribute(CSKYAttrs::CSKY_VDSP_VERSION, CSKYAttrs::VDSP_VERSION_2);
 297: 
 298:   if (STI.hasFeature(CSKY::FeatureFPUV2_SF) ||
 299:       STI.hasFeature(CSKY::FeatureFPUV2_DF))
 300:     emitAttribute(CSKYAttrs::CSKY_FPU_VERSION, CSKYAttrs::FPU_VERSION_2);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 301-320

```cpp
 301:   else if (STI.hasFeature(CSKY::FeatureFPUV3_HF) ||
 302:            STI.hasFeature(CSKY::FeatureFPUV3_SF) ||
 303:            STI.hasFeature(CSKY::FeatureFPUV3_DF))
 304:     emitAttribute(CSKYAttrs::CSKY_FPU_VERSION, CSKYAttrs::FPU_VERSION_3);
 305: 
 306:   bool hasAnyFloatExt = STI.hasFeature(CSKY::FeatureFPUV2_SF) ||
 307:                         STI.hasFeature(CSKY::FeatureFPUV2_DF) ||
 308:                         STI.hasFeature(CSKY::FeatureFPUV3_HF) ||
 309:                         STI.hasFeature(CSKY::FeatureFPUV3_SF) ||
 310:                         STI.hasFeature(CSKY::FeatureFPUV3_DF);
 311: 
 312:   if (hasAnyFloatExt && STI.hasFeature(CSKY::ModeHardFloat) &&
 313:       STI.hasFeature(CSKY::ModeHardFloatABI))
 314:     emitAttribute(CSKYAttrs::CSKY_FPU_ABI, CSKYAttrs::FPU_ABI_HARD);
 315:   else if (hasAnyFloatExt && STI.hasFeature(CSKY::ModeHardFloat))
 316:     emitAttribute(CSKYAttrs::CSKY_FPU_ABI, CSKYAttrs::FPU_ABI_SOFTFP);
 317:   else
 318:     emitAttribute(CSKYAttrs::CSKY_FPU_ABI, CSKYAttrs::FPU_ABI_SOFT);
 319: 
 320:   unsigned HardFPFlag = 0;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 321-336

```cpp
 321:   if (STI.hasFeature(CSKY::FeatureFPUV3_HF))
 322:     HardFPFlag |= CSKYAttrs::FPU_HARDFP_HALF;
 323:   if (STI.hasFeature(CSKY::FeatureFPUV2_SF) ||
 324:       STI.hasFeature(CSKY::FeatureFPUV3_SF))
 325:     HardFPFlag |= CSKYAttrs::FPU_HARDFP_SINGLE;
 326:   if (STI.hasFeature(CSKY::FeatureFPUV2_DF) ||
 327:       STI.hasFeature(CSKY::FeatureFPUV3_DF))
 328:     HardFPFlag |= CSKYAttrs::FPU_HARDFP_DOUBLE;
 329: 
 330:   if (HardFPFlag != 0) {
 331:     emitAttribute(CSKYAttrs::CSKY_FPU_DENORMAL, CSKYAttrs::NEEDED);
 332:     emitAttribute(CSKYAttrs::CSKY_FPU_EXCEPTION, CSKYAttrs::NEEDED);
 333:     emitTextAttribute(CSKYAttrs::CSKY_FPU_NUMBER_MODULE, "IEEE 754");
 334:     emitAttribute(CSKYAttrs::CSKY_FPU_HARDFP, HardFPFlag);
 335:   }
 336: }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

## Key Concepts / 关键概念

- MC streaming and emission / MC 流式输出
- CPU feature modelling / CPU 特性建模
- ELF object support / ELF 目标文件支持

## Dependencies / 依赖关系

- Direct includes / 直接包含: `CSKYELFStreamer.h`, `CSKYMCTargetDesc.h`, `MCTargetDesc/CSKYAsmBackend.h`, `MCTargetDesc/CSKYBaseInfo.h`, `llvm/BinaryFormat/ELF.h`, `llvm/MC/MCAssembler.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCELFObjectWriter.h`
- LLVM subsystems / LLVM 子系统: LLVM MC
- Local companions / 本地配套文件: `CSKYELFStreamer.h`
