# CSKY.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKY.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Acts as the central TableGen description for the target, wiring together registers, instructions, subtargets, and features.
- 目的（中文）: 作为目标的核心 TableGen 描述文件，汇总寄存器、指令、子目标和特性定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===-- CSKY.td - Describe the CSKY Target Machine ---------*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: include "llvm/Target/Target.td"
  10: 
  11: //===----------------------------------------------------------------------===//
  12: // CSKY subtarget features and instruction predicates.
  13: //===----------------------------------------------------------------------===//
  14: def ModeHardFloat :
  15:   SubtargetFeature<"hard-float", "UseHardFloat",
  16:                    "true", "Use hard floating point features">;
  17: def ModeHardFloatABI :
  18:   SubtargetFeature<"hard-float-abi", "UseHardFloatABI",
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. It imports other TableGen fragments so the current description can reuse shared records, register sets, or instruction definitions. This range defines declarative TableGen records such as ModeHardFloat, ModeHardFloatABI, capturing target metadata instead of imperative runtime logic.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 它引入其他 TableGen 片段，以便当前描述复用共享记录、寄存器集合或指令定义。 这一段定义了 ModeHardFloat, ModeHardFloatABI 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 19-36

```tablegen
  19:                    "true", "Use hard floating point ABI to pass args">;
  20: 
  21: def FeatureFPUV2_SF
  22:     : SubtargetFeature<"fpuv2_sf", "HasFPUv2SingleFloat", "true",
  23:                        "Enable FPUv2 single float instructions">;
  24: def HasFPUv2_SF : Predicate<"Subtarget->hasFPUv2SingleFloat()">,
  25:                   AssemblerPredicate<(all_of FeatureFPUV2_SF),
  26:                   "Enable FPUv2 single float instructions">;
  27: 
  28: def FeatureFPUV2_DF
  29:     : SubtargetFeature<"fpuv2_df", "HasFPUv2DoubleFloat", "true",
  30:                        "Enable FPUv2 double float instructions">;
  31: def HasFPUv2_DF : Predicate<"Subtarget->hasFPUv2DoubleFloat()">,
  32:                   AssemblerPredicate<(all_of FeatureFPUV2_DF),
  33:                   "Enable FPUv2 double float instructions">;
  34: 
  35: def FeatureFdivdu : SubtargetFeature<"fdivdu", "HasFdivdu", "true",
  36:                                      "Enable float divide instructions">;
```

- EN: This range defines declarative TableGen records such as FeatureFPUV2_SF, HasFPUv2_SF, FeatureFPUV2_DF, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 FeatureFPUV2_SF, HasFPUv2_SF, FeatureFPUV2_DF 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 37-54

```tablegen
  37: def HasFdivdu : Predicate<"Subtarget->hasFdivdu()">,
  38:                 AssemblerPredicate<(all_of FeatureFdivdu),
  39:                 "Enable float divide instructions">;
  40: 
  41: def FeatureFPUV3_HI
  42:     : SubtargetFeature<"fpuv3_hi", "HasFPUv3HalfWord", "true",
  43:                        "Enable FPUv3 half word converting instructions">;
  44: def HasFPUv3_HI : Predicate<"Subtarget->hasFPUv3HalfWord()">,
  45:                   AssemblerPredicate<(all_of FeatureFPUV3_HI),
  46:                   "Enable FPUv3 half word converting instructions">;
  47: 
  48: def FeatureFPUV3_HF
  49:     : SubtargetFeature<"fpuv3_hf", "HasFPUv3HalfFloat", "true",
  50:                        "Enable FPUv3 half precision operate instructions">;
  51: def HasFPUv3_HF : Predicate<"Subtarget->hasFPUv3HalfFloat()">,
  52:                   AssemblerPredicate<(all_of FeatureFPUV3_HF),
  53:                   "Enable FPUv3 half precision operate instructions">;
  54: 
```

- EN: This range defines declarative TableGen records such as HasFdivdu, FeatureFPUV3_HI, HasFPUv3_HI, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 HasFdivdu, FeatureFPUV3_HI, HasFPUv3_HI 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 55-72

```tablegen
  55: def FeatureFPUV3_SF
  56:     : SubtargetFeature<"fpuv3_sf", "HasFPUv3SingleFloat", "true",
  57:                        "Enable FPUv3 single float instructions">;
  58: def HasFPUv3_SF : Predicate<"Subtarget->hasFPUv3SingleFloat()">,
  59:                   AssemblerPredicate<(all_of FeatureFPUV3_SF),
  60:                   "Enable FPUv3 single float instructions">;
  61: 
  62: def FeatureFPUV3_DF
  63:     : SubtargetFeature<"fpuv3_df", "HasFPUv3DoubleFloat", "true",
  64:                        "Enable FPUv3 double float instructions">;
  65: def HasFPUv3_DF : Predicate<"Subtarget->hasFPUv3DoubleFloat()">,
  66:                   AssemblerPredicate<(all_of FeatureFPUV3_DF),
  67:                   "Enable FPUv3 double float instructions">;
  68: 
  69: def HasFLOATE1
  70:     : SubtargetFeature<"floate1", "HasFLOATE1", "true", "Support CSKY floate1 instructions">;
  71: def iHasFLOATE1 : Predicate<"Subtarget->hasFLOATE1()">,
  72:              AssemblerPredicate<(all_of HasFLOATE1),
```

- EN: This range defines declarative TableGen records such as FeatureFPUV3_SF, HasFPUv3_SF, FeatureFPUV3_DF, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 FeatureFPUV3_SF, HasFPUv3_SF, FeatureFPUV3_DF 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 73-90

```tablegen
  73:              "Support CSKY floate1 instructions">;
  74: 
  75: def HasFLOAT1E2
  76:     : SubtargetFeature<"float1e2", "HasFLOAT1E2", "true", "Support CSKY float1e2 instructions">;
  77: def iHasFLOAT1E2 : Predicate<"Subtarget->hasFLOAT1E2()">,
  78:              AssemblerPredicate<(all_of HasFLOAT1E2),
  79:              "Support CSKY float1e2 instructions">;
  80: 
  81: def HasFLOAT1E3
  82:     : SubtargetFeature<"float1e3", "HasFLOAT1E3", "true", "Support CSKY float1e3 instructions">;
  83: def iHasFLOAT1E3 : Predicate<"Subtarget->hasFLOAT1E3()">,
  84:              AssemblerPredicate<(all_of HasFLOAT1E3),
  85:              "Support CSKY float1e3 instructions">;
  86: 
  87: def HasFLOAT3E4
  88:     : SubtargetFeature<"float3e4", "HasFLOAT3E4", "true", "Support CSKY float3e4 instructions">;
  89: def iHasFLOAT3E4 : Predicate<"Subtarget->hasFLOAT3E4()">,
  90:              AssemblerPredicate<(all_of HasFLOAT3E4),
```

- EN: This range defines declarative TableGen records such as HasFLOAT1E2, iHasFLOAT1E2, HasFLOAT1E3, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 HasFLOAT1E2, iHasFLOAT1E2, HasFLOAT1E3 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 91-108

```tablegen
  91:              "Support CSKY float3e4 instructions">;
  92: 
  93: def HasFLOAT7E60
  94:     : SubtargetFeature<"float7e60", "HasFLOAT7E60", "true", "Support CSKY float7e60 instructions">;
  95: def iHasFLOAT7E60 : Predicate<"Subtarget->hasFLOAT7E60()">,
  96:              AssemblerPredicate<(all_of HasFLOAT7E60),
  97:              "Support CSKY float7e60 instructions">;
  98: 
  99: def FeatureHWDiv : SubtargetFeature<"hwdiv", "HasHardwareDivide", "true",
 100:                                     "Enable divide instructions">;
 101: def HasHWDiv : Predicate<"Subtarget->hasHardwareDivide()">,
 102:                AssemblerPredicate<(all_of FeatureHWDiv),
 103:                "Enable divide instructions">;
 104: 
 105: def FeatureSTM : SubtargetFeature<"multiple_stld", "HasSTM", "true",
 106:                                   "Enable multiple load/store instructions">;
 107: def HasSTM : Predicate<"Subtarget->hasSTM()">,
 108:              AssemblerPredicate<(all_of FeatureSTM),
```

- EN: This range defines declarative TableGen records such as HasFLOAT7E60, iHasFLOAT7E60, FeatureHWDiv, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 HasFLOAT7E60, iHasFLOAT7E60, FeatureHWDiv 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 109-126

```tablegen
 109:              "Enable multiple load/store instructions">;
 110: 
 111: def FeaturePushPop : SubtargetFeature<"pushpop", "HasPushPop", "true",
 112:                                       "Enable push/pop instructions">;
 113: def HasPushPop : Predicate<"Subtarget->hasPushPop()">,
 114:                  AssemblerPredicate<(all_of FeaturePushPop),
 115:                  "Enable push/pop instructions">;
 116: 
 117: def FeatureDSP
 118:     : SubtargetFeature<"edsp", "HasDSP", "true", "Enable DSP instructions">;
 119: def HasDSP : Predicate<"Subtarget->hasDSP()">,
 120:              AssemblerPredicate<(all_of FeatureDSP),
 121:              "Enable DSP instructions">;
 122: 
 123: def HasDSP1E2
 124:     : SubtargetFeature<"dsp1e2", "HasDSP1E2", "true", "Support CSKY dsp1e2 instructions">;
 125: def iHasDSP1E2 : Predicate<"Subtarget->hasDSP1E2()">,
 126:              AssemblerPredicate<(all_of HasDSP1E2),
```

- EN: This range defines declarative TableGen records such as FeaturePushPop, HasPushPop, FeatureDSP, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 FeaturePushPop, HasPushPop, FeatureDSP 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 127-144

```tablegen
 127:              "Support CSKY dsp1e2 instructions">;
 128: 
 129: def HasDSPE60
 130:     : SubtargetFeature<"dspe60", "HasDSPE60", "true", "Support CSKY dspe60 instructions">;
 131: def iHasDSPE60 : Predicate<"Subtarget->hasDSPE60()">,
 132:              AssemblerPredicate<(all_of HasDSPE60),
 133:              "Support CSKY dspe60 instructions">;
 134: 
 135: def FeatureDSPV2 : SubtargetFeature<"dspv2", "HasDSPV2", "true",
 136:                                     "Enable DSP V2.0 instructions">;
 137: def HasDSPV2 : Predicate<"Subtarget->hasDSPV2()">,
 138:                AssemblerPredicate<(all_of FeatureDSPV2),
 139:                "Enable DSP V2.0 instructions">;
 140: 
 141: def FeatureDSP_Silan : SubtargetFeature<"dsp_silan", "HasDSP_Silan", "true",
 142:                                     "Enable DSP Silan instructions">;
 143: def HasDSP_Silan : Predicate<"Subtarget->hasDSP_Silan()">,
 144:                AssemblerPredicate<(all_of FeatureDSP_Silan),
```

- EN: This range defines declarative TableGen records such as HasDSPE60, iHasDSPE60, FeatureDSPV2, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 HasDSPE60, iHasDSPE60, FeatureDSPV2 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 145-162

```tablegen
 145:                "Enable DSP Silan instructions">;
 146: 
 147: // Atomic Support
 148: def FeatureBTST16 : SubtargetFeature<"btst16", "HasBTST16", "true",
 149:                                      "Use the 16-bit btsti instruction">;
 150: def HasBTST16 : Predicate<"Subtarget->hasBTST16()">,
 151:                    AssemblerPredicate<(all_of FeatureBTST16),
 152:                    "Use the 16-bit btsti instruction">;
 153: 
 154: // Atomic Support
 155: def FeatureExtendLrw : SubtargetFeature<"elrw", "HasExtendLrw", "true",
 156:                                         "Use the extend LRW instruction">;
 157: def HasExtendLrw : Predicate<"Subtarget->hasExtendLrw()">,
 158:                    AssemblerPredicate<(all_of FeatureExtendLrw),
 159:                    "Use the extend LRW instruction">;
 160: 
 161: def FeatureTrust : SubtargetFeature<"trust", "HasTrust", "true",
 162:                                     "Enable trust instructions">;
```

- EN: This range defines declarative TableGen records such as FeatureBTST16, HasBTST16, FeatureExtendLrw, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 FeatureBTST16, HasBTST16, FeatureExtendLrw 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 163-180

```tablegen
 163: def HasTrust : Predicate<"Subtarget->hasTrust()">,
 164:                AssemblerPredicate<(all_of FeatureTrust),
 165:                "Enable trust instructions">;
 166: 
 167: def FeatureJAVA
 168:     : SubtargetFeature<"java", "HasJAVA", "true", "Enable java instructions">;
 169: def HasJAVA : Predicate<"Subtarget->hasJAVA()">,
 170:               AssemblerPredicate<(all_of FeatureJAVA),
 171:               "Enable java instructions">;
 172: 
 173: def FeatureCache
 174:     : SubtargetFeature<"cache", "HasCache", "true", "Enable cache">;
 175: def HasCache : Predicate<"Subtarget->hasCache()">,
 176:               AssemblerPredicate<(all_of FeatureCache),
 177:               "Enable cache">;
 178: 
 179: def FeatureNVIC
 180:     : SubtargetFeature<"nvic", "HasNVIC", "true", "Enable NVIC">;
```

- EN: This range defines declarative TableGen records such as HasTrust, FeatureJAVA, HasJAVA, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 HasTrust, FeatureJAVA, HasJAVA 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 181-198

```tablegen
 181: def HasNVIC : Predicate<"Subtarget->hasNVIC()">,
 182:               AssemblerPredicate<(all_of FeatureNVIC),
 183:               "Enable NVIC">;
 184: 
 185: def FeatureDoloop : SubtargetFeature<"doloop", "HasDoloop", "true",
 186:                                      "Enable doloop instructions">;
 187: def HasDoloop : Predicate<"Subtarget->hasDoloop()">,
 188:                 AssemblerPredicate<(all_of FeatureDoloop),
 189:                 "Enable doloop instructions">;
 190: 
 191: // Other features than instructions
 192: def FeatureHighreg : SubtargetFeature<"high-registers", "HasHighRegisters",
 193:                                       "true", "Enable r16-r31 registers">;
 194: def HasHighRegisters : Predicate<"Subtarget->hasHighRegisters()">,
 195:                        AssemblerPredicate<(all_of FeatureHighreg),
 196:                        "Enable r16-r31 registers">;
 197: 
 198: def FeatureSmart : SubtargetFeature<"smart", "SmartMode", "true",
```

- EN: This range defines declarative TableGen records such as HasNVIC, FeatureDoloop, HasDoloop, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 HasNVIC, FeatureDoloop, HasDoloop 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 199-216

```tablegen
 199:                                     "Let CPU work in Smart Mode">;
 200: def SmartMode : Predicate<"Subtarget->smartMode()">,
 201:                 AssemblerPredicate<(all_of FeatureSmart),
 202:                 "Let CPU work in Smart Mode">;
 203: 
 204: def FeatureVDSPV2 : SubtargetFeature<"vdspv2", "HasVDSPV2", "true",
 205:                                      "Enable vdsp-v2 instructions">;
 206: def HasVDSPV2 : Predicate<"Subtarget->hasVDSPV2()">,
 207:                 AssemblerPredicate<(all_of FeatureVDSPV2),
 208:                 "Enable vdsp-v2 instructions">;
 209: 
 210: def HasVDSPV2_FLOAT : Predicate<"Subtarget->hasVDSPV2_FLOAT()">;
 211: def HasVDSPV2_HALF: Predicate<"Subtarget->hasVDSPV2_HALF()">;
 212: 
 213: def HasVDSP2E3
 214:     : SubtargetFeature<"vdsp2e3", "HasVDSP2E3", "true", "Support CSKY vdsp2e3 instructions">;
 215: def iHasVDSP2E3 : Predicate<"Subtarget->hasVDSP2E3()">,
 216:              AssemblerPredicate<(all_of HasVDSP2E3),
```

- EN: This range defines declarative TableGen records such as SmartMode, FeatureVDSPV2, HasVDSPV2, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 SmartMode, FeatureVDSPV2, HasVDSPV2 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 217-234

```tablegen
 217:              "Support CSKY vdsp2e3 instructions">;
 218: 
 219: def HasVDSP2E60F
 220:     : SubtargetFeature<"vdsp2e60f", "HasVDSP2E60F", "true", "Support CSKY vdsp2e60f instructions">;
 221: def iHasVDSP2E60F : Predicate<"Subtarget->hasVDSP2E60F()">,
 222:              AssemblerPredicate<(all_of HasVDSP2E60F),
 223:              "Support CSKY vdsp2e60f instructions">;
 224: 
 225: def FeatureHardTP : SubtargetFeature<"hard-tp", "ReadTPHard", "true",
 226:                                      "Enable TLS Pointer register">;
 227: def ReadTPHard : Predicate<"Subtarget->readTPHard()">,
 228:                  AssemblerPredicate<(all_of FeatureHardTP),
 229:                  "Enable TLS Pointer register">;
 230: 
 231: def FeatureSoftTP : SubtargetFeature<"soft-tp", "ReadTPHard", "false",
 232:                                      "Disable TLS Pointer register">;
 233: 
 234: def FeatureIstack : SubtargetFeature<"istack", "EnableInterruptAttribute",
```

- EN: This range defines declarative TableGen records such as HasVDSP2E60F, iHasVDSP2E60F, FeatureHardTP, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 HasVDSP2E60F, iHasVDSP2E60F, FeatureHardTP 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 235-252

```tablegen
 235:                                      "true", "Enable interrupt attribute">;
 236: def EnableInterruptAttribute
 237:     : Predicate<"Subtarget->enableInterruptAttribute()">,
 238:       AssemblerPredicate<(all_of FeatureIstack),
 239:       "Enable interrupt attribute">;
 240: 
 241: def FeatureConstPool : SubtargetFeature<"constpool", "DumpConstPool", "true",
 242:                                         "Dump the constant pool by compiler">;
 243: def DumpConstPool : Predicate<"Subtarget->dumpConstPool()">,
 244:                     AssemblerPredicate<(all_of FeatureConstPool),
 245:                     "Dump the constant pool by compiler">;
 246: 
 247: def FeatureStackSize : SubtargetFeature<"stack-size", "EnableStackSize", "true",
 248:                                         "Output stack size information">;
 249: def EnableStackSize : Predicate<"Subtarget->enableStackSize()">,
 250:                       AssemblerPredicate<(all_of FeatureStackSize),
 251:                       "Output stack size information">;
 252: 
```

- EN: This range defines declarative TableGen records such as EnableInterruptAttribute, FeatureConstPool, DumpConstPool, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 EnableInterruptAttribute, FeatureConstPool, DumpConstPool 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 253-270

```tablegen
 253: def FeatureCCRT
 254:     : SubtargetFeature<"ccrt", "UseCCRT", "true", "Use CSKY compiler runtime">;
 255: def UseCCRT : Predicate<"Subtarget->useCCRT()">,
 256:               AssemblerPredicate<(all_of FeatureCCRT),
 257:               "Use CSKY compiler runtime">;
 258: 
 259: def FeatureVDSPV1_128 : SubtargetFeature<"vdspv1", "HasVDSPV1_128", "true",
 260:                                          "Enable 128bit vdsp-v1 instructions">;
 261: def HasVDSPV1_128 : Predicate<"Subtarget->hasVDSPV1_128()">,
 262:                     AssemblerPredicate<(all_of FeatureVDSPV1_128),
 263:                     "Enable 128bit vdsp-v1 instructions">;
 264: 
 265: def HasE1
 266:     : SubtargetFeature<"e1", "HasE1", "true", "Support CSKY e1 instructions",
 267:                        [FeatureExtendLrw]>;
 268: def iHasE1 : Predicate<"Subtarget->hasE1()">,
 269:              AssemblerPredicate<(all_of HasE1),
 270:              "Support CSKY e1 instructions">;
```

- EN: This range defines declarative TableGen records such as FeatureCCRT, UseCCRT, FeatureVDSPV1_128, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 FeatureCCRT, UseCCRT, FeatureVDSPV1_128 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 271-288

```tablegen
 271: 
 272: def HasE2
 273:     : SubtargetFeature<"e2", "HasE2", "true", "Support CSKY e2 instructions",
 274:                        [HasE1]>;
 275: def iHasE2 : Predicate<"Subtarget->hasE2()">,
 276:              AssemblerPredicate<(all_of HasE2),
 277:              "Support CSKY e2 instructions">;
 278: 
 279: def Has2E3 : SubtargetFeature<"2e3", "Has2E3", "true",
 280:                               "Support CSKY 2e3 instructions", [HasE2]>;
 281: def iHas2E3 : Predicate<"Subtarget->has2E3()">,
 282:               AssemblerPredicate<(all_of Has2E3),
 283:               "Support CSKY 2e3 instructions">;
 284: 
 285: def HasMP : SubtargetFeature<"mp", "HasMP", "true",
 286:                                 "Support CSKY mp instructions", [Has2E3]>;
 287: def iHasMP : Predicate<"Subtarget->hasMP()">,
 288:                 AssemblerPredicate<(all_of HasMP),
```

- EN: This range defines declarative TableGen records such as HasE2, iHasE2, Has2E3, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 HasE2, iHasE2, Has2E3 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 289-306

```tablegen
 289:                 "Support CSKY mp instructions">;
 290: 
 291: def Has3E3r1 : SubtargetFeature<"3e3r1", "Has3E3r1", "true",
 292:                                 "Support CSKY 3e3r1 instructions">;
 293: def iHas3E3r1 : Predicate<"Subtarget->has3E3r1()">,
 294:                 AssemblerPredicate<(all_of Has3E3r1),
 295:                 "Support CSKY 3e3r1 instructions">;
 296: 
 297: def Has3r1E3r2 : SubtargetFeature<"3e3r2", "Has3r1E3r2", "true",
 298:                                   "Support CSKY 3e3r2 instructions",
 299:                                   [Has3E3r1, FeatureDoloop]>;
 300: def iHas3r1E3r2 : Predicate<"Subtarget->has3r1E3r2()">,
 301:                   AssemblerPredicate<(all_of Has3r1E3r2),
 302:                   "Support CSKY 3e3r2 instructions">;
 303: 
 304: def Has3r2E3r3
 305:     : SubtargetFeature<"3e3r3", "Has3r2E3r3", "true",
 306:                        "Support CSKY 3e3r3 instructions", [FeatureDoloop]>;
```

- EN: This range defines declarative TableGen records such as Has3E3r1, iHas3E3r1, Has3r1E3r2, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 Has3E3r1, iHas3E3r1, Has3r1E3r2 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 307-324

```tablegen
 307: def iHas3r2E3r3 : Predicate<"Subtarget->has3r2E3r3()">,
 308:                   AssemblerPredicate<(all_of Has3r2E3r3),
 309:                   "Support CSKY 3e3r3 instructions">;
 310: 
 311: def Has3E7 : SubtargetFeature<"3e7", "Has3E7", "true",
 312:                               "Support CSKY 3e7 instructions", [Has2E3]>;
 313: def iHas3E7 : Predicate<"Subtarget->has3E7()">,
 314:               AssemblerPredicate<(all_of Has3E7),
 315:               "Support CSKY 3e7 instructions">;
 316: 
 317: def HasMP1E2 : SubtargetFeature<"mp1e2", "HasMP1E2", "true",
 318:                                 "Support CSKY mp1e2 instructions", [Has3E7]>;
 319: def iHasMP1E2 : Predicate<"Subtarget->hasMP1E2()">,
 320:                 AssemblerPredicate<(all_of HasMP1E2),
 321:                 "Support CSKY mp1e2 instructions">;
 322: 
 323: def Has7E10 : SubtargetFeature<"7e10", "Has7E10", "true",
 324:                                "Support CSKY 7e10 instructions", [Has3E7]>;
```

- EN: This range defines declarative TableGen records such as iHas3r2E3r3, Has3E7, iHas3E7, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 iHas3r2E3r3, Has3E7, iHas3E7 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 325-342

```tablegen
 325: def iHas7E10 : Predicate<"Subtarget->has7E10()">,
 326:                AssemblerPredicate<(all_of Has7E10),
 327:                "Support CSKY 7e10 instructions">;
 328: 
 329: def Has10E60 : SubtargetFeature<"10e60", "Has10E60", "true",
 330:                                "Support CSKY 10e60 instructions", [Has7E10]>;
 331: def iHas10E60 : Predicate<"Subtarget->has10E60()">,
 332:                AssemblerPredicate<(all_of Has10E60),
 333:                "Support CSKY 10e60 instructions">;
 334: 
 335: //===----------------------------------------------------------------------===//
 336: // CSKY Processor subtarget features.
 337: //===----------------------------------------------------------------------===//
 338: 
 339: def ProcCK801 : SubtargetFeature<"ck801", "CSKYProcFamily", "CK801",
 340:                                  "CSKY ck801 processors", []>;
 341: def isCK801 : Predicate<"Subtarget->isCK801()">,
 342:               AssemblerPredicate<(all_of ProcCK801)>;
```

- EN: This range defines declarative TableGen records such as iHas7E10, Has10E60, iHas10E60, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 iHas7E10, Has10E60, iHas10E60 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 343-360

```tablegen
 343: def ProcCK802 : SubtargetFeature<"ck802", "CSKYProcFamily", "CK802",
 344:                                  "CSKY ck802 processors", []>;
 345: def ProcCK803 : SubtargetFeature<"ck803", "CSKYProcFamily", "CK803",
 346:                                  "CSKY ck803 processors", []>;
 347: def ProcCK803S : SubtargetFeature<"ck803s", "CSKYProcFamily", "CK803S",
 348:                                  "CSKY ck803s processors", []>;
 349: def ProcCK804 : SubtargetFeature<"ck804", "CSKYProcFamily", "CK804",
 350:                                  "CSKY ck804 processors", []>;
 351: def ProcCK805 : SubtargetFeature<"ck805", "CSKYProcFamily", "CK805",
 352:                                  "CSKY ck805 processors", []>;
 353: def ProcCK807 : SubtargetFeature<"ck807", "CSKYProcFamily", "CK807",
 354:                                  "CSKY ck807 processors", []>;
 355: def ProcCK810 : SubtargetFeature<"ck810", "CSKYProcFamily", "CK810",
 356:                                  "CSKY ck810 processors", []>;
 357: def ProcCK810V : SubtargetFeature<"ck810v", "CSKYProcFamily", "CK810V",
 358:                                  "CSKY ck810v processors", []>;
 359: def ProcCK860 : SubtargetFeature<"ck860", "CSKYProcFamily", "CK860",
 360:                                  "CSKY ck860 processors", []>;
```

- EN: This range defines declarative TableGen records such as ProcCK802, ProcCK803, ProcCK803S, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 ProcCK802, ProcCK803, ProcCK803S 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 361-378

```tablegen
 361: def ProcCK860V : SubtargetFeature<"ck860v", "CSKYProcFamily", "CK860V",
 362:                                  "CSKY ck860v processors", []>;
 363: 
 364: //===----------------------------------------------------------------------===//
 365: // Registers, calling conventions, instruction descriptions.
 366: //===----------------------------------------------------------------------===//
 367: 
 368: include "CSKYRegisterInfo.td"
 369: include "CSKYCallingConv.td"
 370: include "CSKYInstrInfo.td"
 371: 
 372: //===----------------------------------------------------------------------===//
 373: // CSKY processors supported.
 374: //===----------------------------------------------------------------------===//
 375: 
 376: // btsti16 is one of most basic instructions should be enable
 377: // even in generic processor to avoid failure codegen.
 378: def : ProcessorModel<"generic", NoSchedModel, [FeatureBTST16]>;
```

- EN: It imports other TableGen fragments so the current description can reuse shared records, register sets, or instruction definitions. This range defines declarative TableGen records such as ProcCK860V, capturing target metadata instead of imperative runtime logic.
- 中文: 它引入其他 TableGen 片段，以便当前描述复用共享记录、寄存器集合或指令定义。 这一段定义了 ProcCK860V 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 379-396

```tablegen
 379: 
 380: // CK801 series
 381: class CK801<string n, SchedMachineModel m, list<SubtargetFeature> f,
 382:             list<SubtargetFeature> tunef = []>
 383:     : ProcessorModel<n, m, !listconcat(f, [HasE1, FeatureTrust, FeatureBTST16, ProcCK801]), !listconcat(tunef, [])>;
 384: 
 385: def : CK801<"ck801", NoSchedModel, []>;
 386: def : CK801<"ck801t", NoSchedModel, []>;
 387: def : CK801<"e801", NoSchedModel, []>;
 388: 
 389: // CK802 series
 390: class CK802<string n, SchedMachineModel m, list<SubtargetFeature> f,
 391:             list<SubtargetFeature> tunef = []>
 392:     : ProcessorModel<n, m, !listconcat(f, [HasE2, FeatureTrust, FeatureBTST16, FeatureNVIC, ProcCK802]), !listconcat(tunef, [])>;
 393: 
 394: def : CK802<"ck802", NoSchedModel, []>;
 395: def : CK802<"ck802t", NoSchedModel, []>;
 396: def : CK802<"ck802j", NoSchedModel, [FeatureJAVA]>;
```

- EN: This range defines declarative TableGen records such as CK801, CK802, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 CK801, CK802 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 397-414

```tablegen
 397: def : CK802<"e802", NoSchedModel, []>;
 398: def : CK802<"e802t", NoSchedModel, []>;
 399: def : CK802<"s802", NoSchedModel, []>;
 400: def : CK802<"s802t", NoSchedModel, []>;
 401: 
 402: // CK803 series
 403: class CK803<string n, SchedMachineModel m, list<SubtargetFeature> f,
 404:             list<SubtargetFeature> tunef = []>
 405:     : ProcessorModel<n, m, !listconcat(f, [Has2E3, HasMP, FeatureTrust, FeatureBTST16, FeatureNVIC, FeatureHWDiv, ProcCK803]), !listconcat(tunef, [])>;
 406: 
 407: def : CK803<"ck803", NoSchedModel, []>;
 408: def : CK803<"ck803h", NoSchedModel, []>;
 409: def : CK803<"ck803t", NoSchedModel, []>;
 410: def : CK803<"ck803ht", NoSchedModel, []>;
 411: def : CK803<"ck803f", NoSchedModel, [FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
 412: def : CK803<"ck803fh", NoSchedModel, [FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
 413: def : CK803<"ck803e", NoSchedModel, [FeatureDSP, HasDSP1E2, HasDSPE60]>;
 414: def : CK803<"ck803eh", NoSchedModel, [FeatureDSP, HasDSP1E2, HasDSPE60]>;
```

- EN: This range defines declarative TableGen records such as CK803, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 CK803 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 415-432

```tablegen
 415: def : CK803<"ck803et", NoSchedModel, [FeatureDSP, HasDSP1E2, HasDSPE60]>;
 416: def : CK803<"ck803eht", NoSchedModel, [FeatureDSP, HasDSP1E2, HasDSPE60]>;
 417: def : CK803<"ck803ef", NoSchedModel,
 418:             [FeatureDSP, HasDSP1E2, HasDSPE60, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
 419: def : CK803<"ck803efh", NoSchedModel,
 420:             [FeatureDSP, HasDSP1E2, HasDSPE60, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
 421: def : CK803<"ck803ft", NoSchedModel, [FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
 422: def : CK803<"ck803eft", NoSchedModel,
 423:             [FeatureDSP, HasDSP1E2, HasDSPE60, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
 424: def : CK803<"ck803efht", NoSchedModel,
 425:             [FeatureDSP, HasDSP1E2, HasDSPE60, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
 426: def : CK803<"ck803r1", NoSchedModel, [Has3E3r1, Has3r2E3r3, FeatureDSPV2]>;
 427: def : CK803<"ck803hr1", NoSchedModel, [Has3E3r1, Has3r2E3r3, FeatureDSPV2]>;
 428: def : CK803<"ck803tr1", NoSchedModel, [Has3E3r1, Has3r2E3r3, FeatureDSPV2]>;
 429: def : CK803<"ck803htr1", NoSchedModel, [Has3E3r1, Has3r2E3r3, FeatureDSPV2]>;
 430: def : CK803<"ck803fr1", NoSchedModel,
 431:             [Has3E3r1, Has3r2E3r3, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3, FeatureDSPV2]>;
 432: def : CK803<"ck803fhr1", NoSchedModel,
```

- EN: This range continues the implementation of the backend component described by CSKY.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 433-450

```tablegen
 433:             [Has3E3r1, Has3r2E3r3, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3, FeatureDSPV2]>;
 434: def : CK803<"ck803er1", NoSchedModel,
 435:             [Has3E3r1, Has3r2E3r3, FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSPV2, FeatureHighreg]>;
 436: def : CK803<"ck803etr1", NoSchedModel,
 437:             [Has3E3r1, Has3r2E3r3, FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSPV2, FeatureHighreg]>;
 438: def : CK803<"ck803ehr1", NoSchedModel,
 439:             [Has3E3r1, Has3r2E3r3, FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSPV2, FeatureHighreg]>;
 440: def : CK803<"ck803ehtr1", NoSchedModel,
 441:             [Has3E3r1, Has3r2E3r3, FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSPV2, FeatureHighreg]>;
 442: def : CK803<"ck803efr1", NoSchedModel,
 443:             [Has3E3r1, FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3, FeatureHighreg]>;
 444: def : CK803<"ck803efhr1", NoSchedModel,
 445:             [Has3E3r1, FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3, FeatureHighreg]>;
 446: def : CK803<"ck803ftr1", NoSchedModel, [Has3E3r1, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3, FeatureDSPV2]>;
 447: def : CK803<"ck803eftr1", NoSchedModel,
 448:             [Has3E3r1, FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3, FeatureHighreg]>;
 449: def : CK803<"ck803efhtr1", NoSchedModel,
 450:             [Has3E3r1, FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3, FeatureHighreg]>;
```

- EN: This range continues the implementation of the backend component described by CSKY.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 451-468

```tablegen
 451: def : CK803<"ck803r2", NoSchedModel, [Has3r1E3r2, Has3r2E3r3, FeatureDSPV2]>;
 452: def : CK803<"ck803hr2", NoSchedModel, [Has3r1E3r2, Has3r2E3r3, FeatureDSPV2]>;
 453: def : CK803<"ck803tr2", NoSchedModel, [Has3r1E3r2, Has3r2E3r3, FeatureDSPV2]>;
 454: def : CK803<"ck803htr2", NoSchedModel, [Has3r1E3r2, Has3r2E3r3, FeatureDSPV2]>;
 455: def : CK803<"ck803fr2", NoSchedModel, [Has3r1E3r2, Has3r2E3r3, FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
 456: def : CK803<"ck803fhr2", NoSchedModel, [Has3r1E3r2, Has3r2E3r3, FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
 457: def : CK803<"ck803er2", NoSchedModel, [Has3r1E3r2, Has3r2E3r3, FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSPV2, FeatureHighreg]>;
 458: def : CK803<"ck803etr2", NoSchedModel, [Has3r1E3r2, Has3r2E3r3, FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSPV2, FeatureHighreg]>;
 459: def : CK803<"ck803ehr2", NoSchedModel, [Has3r1E3r2, Has3r2E3r3, FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSPV2, FeatureHighreg]>;
 460: def : CK803<"ck803ehtr2", NoSchedModel, [Has3r1E3r2, Has3r2E3r3, FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSPV2, FeatureHighreg]>;
 461: def : CK803<"ck803efr2", NoSchedModel,
 462:             [Has3r1E3r2, Has3r2E3r3, FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3, FeatureHighreg]>;
 463: def : CK803<"ck803efhr2", NoSchedModel,
 464:             [Has3r1E3r2, Has3r2E3r3, FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3, FeatureHighreg]>;
 465: def : CK803<"ck803ftr2", NoSchedModel, [Has3r1E3r2, Has3r2E3r3, FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
 466: def : CK803<"ck803eftr2", NoSchedModel,
 467:             [Has3r1E3r2, Has3r2E3r3, FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3, FeatureHighreg]>;
 468: def : CK803<"ck803efhtr2", NoSchedModel,
```

- EN: This range continues the implementation of the backend component described by CSKY.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 469-486

```tablegen
 469:             [Has3r1E3r2, Has3r2E3r3, FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3, FeatureHighreg]>;
 470: def : CK803<"ck803r3", NoSchedModel, [Has3r1E3r2, Has3r2E3r3, FeatureDSPV2]>;
 471: def : CK803<"ck803hr3", NoSchedModel, [Has3r1E3r2, Has3r2E3r3, FeatureDSPV2]>;
 472: def : CK803<"ck803tr3", NoSchedModel, [Has3r1E3r2, Has3r2E3r3, FeatureDSPV2]>;
 473: def : CK803<"ck803htr3", NoSchedModel, [Has3r1E3r2, Has3r2E3r3, FeatureDSPV2]>;
 474: def : CK803<"ck803fr3", NoSchedModel, [Has3r1E3r2, Has3r2E3r3, FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
 475: def : CK803<"ck803fhr3", NoSchedModel, [Has3r1E3r2, Has3r2E3r3, FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
 476: def : CK803<"ck803er3", NoSchedModel, [Has3r1E3r2, Has3r2E3r3, FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSPV2, FeatureHighreg]>;
 477: def : CK803<"ck803etr3", NoSchedModel, [Has3r1E3r2, Has3r2E3r3, FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSPV2, FeatureHighreg]>;
 478: def : CK803<"ck803ehr3", NoSchedModel, [Has3r1E3r2, Has3r2E3r3, FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSPV2, FeatureHighreg]>;
 479: def : CK803<"ck803ehtr3", NoSchedModel, [Has3r1E3r2, Has3r2E3r3, FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSPV2, FeatureHighreg]>;
 480: def : CK803<"ck803efr3", NoSchedModel,
 481:             [Has3r1E3r2, Has3r2E3r3, FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3, FeatureHighreg]>;
 482: def : CK803<"ck803efhr3", NoSchedModel,
 483:             [Has3r1E3r2, Has3r2E3r3, FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3, FeatureHighreg]>;
 484: def : CK803<"ck803ftr3", NoSchedModel, [Has3r1E3r2, Has3r2E3r3, FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
 485: def : CK803<"ck803eftr3", NoSchedModel,
 486:             [Has3r1E3r2, Has3r2E3r3, FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3, FeatureHighreg]>;
```

- EN: This range continues the implementation of the backend component described by CSKY.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 487-504

```tablegen
 487: def : CK803<"ck803efhtr3", NoSchedModel,
 488:             [Has3r1E3r2, Has3r2E3r3, FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3, FeatureHighreg]>;
 489: def : CK803<"s803", NoSchedModel, [Has3r1E3r2, Has3r2E3r3]>;
 490: def : CK803<"s803t", NoSchedModel, [Has3r1E3r2, Has3r2E3r3]>;
 491: def : CK803<"e803", NoSchedModel, [Has3r1E3r2, Has3r2E3r3]>;
 492: def : CK803<"e803t", NoSchedModel, [Has3r1E3r2, Has3r2E3r3]>;
 493: 
 494: // CK803S series
 495: class CK803S<string n, SchedMachineModel m, list<SubtargetFeature> f,
 496: list<SubtargetFeature> tunef = []> : CK803<n, m, !listconcat(f, [Has3E3r1, ProcCK803S]), tunef>;
 497: 
 498: def : CK803S<"ck803s", NoSchedModel, []>;
 499: def : CK803S<"ck803sn", NoSchedModel, [FeatureDSP_Silan]>;
 500: def : CK803S<"ck803st", NoSchedModel, []>;
 501: def : CK803S<"ck803snt", NoSchedModel, [FeatureDSP_Silan]>;
 502: def : CK803S<"ck803sf", NoSchedModel, [FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
 503: def : CK803S<"ck803sfn", NoSchedModel, [FeatureFPUV2_SF, FeatureDSP_Silan, HasFLOATE1, HasFLOAT1E3]>;
 504: def : CK803S<"ck803se", NoSchedModel, [FeatureDSP, HasDSP1E2, HasDSPE60]>;
```

- EN: This range defines declarative TableGen records such as CK803S, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 CK803S 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 505-522

```tablegen
 505: def : CK803S<"ck803sen", NoSchedModel, [FeatureDSP, HasDSP1E2, HasDSPE60, FeatureDSP_Silan]>;
 506: def : CK803S<"ck803sef", NoSchedModel,
 507:              [FeatureDSP, HasDSP1E2, HasDSPE60, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
 508: def : CK803S<"ck803sefn", NoSchedModel,
 509:              [FeatureDSP, HasDSP1E2, HasDSPE60, FeatureFPUV2_SF, FeatureDSP_Silan,
 510:               HasFLOATE1, HasFLOAT1E3]>;
 511: def : CK803S<"ck803seft", NoSchedModel,
 512:              [FeatureDSP, HasDSP1E2, HasDSPE60, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
 513: def : CK803S<"ck803sefnt", NoSchedModel,
 514:              [FeatureDSP, HasDSP1E2, HasDSPE60, FeatureFPUV2_SF, FeatureDSP_Silan,
 515:               HasFLOATE1, HasFLOAT1E3]>;
 516: 
 517: // CK804 series
 518: class CK804<string n, SchedMachineModel m, list<SubtargetFeature> f,
 519:             list<SubtargetFeature> tunef = []>
 520:     : CK803<n, m, !listconcat(f, [Has3r1E3r2, Has3r2E3r3, ProcCK804]), !listconcat(tunef, [])>;
 521: 
 522: def : CK804<"ck804", NoSchedModel, []>;
```

- EN: This range defines declarative TableGen records such as CK804, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 CK804 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 523-540

```tablegen
 523: def : CK804<"ck804h", NoSchedModel, []>;
 524: def : CK804<"ck804t", NoSchedModel, []>;
 525: def : CK804<"ck804ht", NoSchedModel, []>;
 526: def : CK804<"ck804f", NoSchedModel, [FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
 527: def : CK804<"ck804fh", NoSchedModel, [FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
 528: def : CK804<"ck804e", NoSchedModel, [FeatureDSPV2, FeatureHighreg]>;
 529: def : CK804<"ck804et", NoSchedModel, [FeatureDSPV2, FeatureHighreg]>;
 530: def : CK804<"ck804eh", NoSchedModel, [FeatureDSPV2, FeatureHighreg]>;
 531: def : CK804<"ck804eht", NoSchedModel, [FeatureDSPV2, FeatureHighreg]>;
 532: def : CK804<"ck804ef", NoSchedModel, [FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3, FeatureHighreg]>;
 533: def : CK804<"ck804efh", NoSchedModel, [FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3, FeatureHighreg]>;
 534: def : CK804<"ck804ft", NoSchedModel, [FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
 535: def : CK804<"ck804eft", NoSchedModel, [FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3, FeatureHighreg]>;
 536: def : CK804<"ck804efht", NoSchedModel, [FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3, FeatureHighreg]>;
 537: def : CK804<"e804d", NoSchedModel, [FeatureDSPV2, FeatureHighreg]>;
 538: def : CK804<"e804dt", NoSchedModel, [FeatureDSPV2, FeatureHighreg]>;
 539: def : CK804<"e804f", NoSchedModel, [FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
 540: def : CK804<"e804ft", NoSchedModel, [FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
```

- EN: This range continues the implementation of the backend component described by CSKY.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 541-558

```tablegen
 541: def : CK804<"e804df", NoSchedModel, [FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3, FeatureHighreg]>;
 542: def : CK804<"e804dft", NoSchedModel, [FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3, FeatureHighreg]>;
 543: 
 544: // CK805 series
 545: class CK805<string n, SchedMachineModel m, list<SubtargetFeature> f,
 546:             list<SubtargetFeature> tunef = []>
 547:     : CK803<n, m, !listconcat(f, [FeatureHighreg, FeatureVDSPV2, HasVDSP2E3, Has3r1E3r2, Has3r2E3r3, ProcCK805]),
 548:             !listconcat(tunef, [])>;
 549: 
 550: def : CK805<"ck805", NoSchedModel, []>;
 551: def : CK805<"i805", NoSchedModel, []>;
 552: def : CK805<"ck805t", NoSchedModel, []>;
 553: def : CK805<"i805f", NoSchedModel, [FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
 554: def : CK805<"ck805f", NoSchedModel, [FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
 555: def : CK805<"ck805e", NoSchedModel, [FeatureDSPV2]>;
 556: def : CK805<"ck805ef", NoSchedModel, [FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
 557: def : CK805<"ck805et", NoSchedModel, [FeatureDSPV2]>;
 558: def : CK805<"ck805ft", NoSchedModel, [FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
```

- EN: This range defines declarative TableGen records such as CK805, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 CK805 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 559-576

```tablegen
 559: def : CK805<"ck805eft", NoSchedModel, [FeatureDSPV2, FeatureFPUV2_SF, HasFLOATE1, HasFLOAT1E3]>;
 560: 
 561: // CK807 series
 562: class CK807<string n, SchedMachineModel m, list<SubtargetFeature> f,
 563:             list<SubtargetFeature> tunef = []>
 564:     : ProcessorModel<n, m, !listconcat(f,
 565:     [ Has3E7, HasMP, HasMP1E2, FeatureTrust, FeatureHWDiv, FeatureDSP, HasDSP1E2, HasDSPE60,
 566:       FeatureHighreg, FeatureHardTP, FeatureNVIC, FeatureCache, ProcCK807]), !listconcat(tunef, [])>;
 567: 
 568: def : CK807<"ck807", NoSchedModel, []>;
 569: def : CK807<"c807", NoSchedModel, []>;
 570: def : CK807<"r807", NoSchedModel, []>;
 571: def : CK807<"ck807e", NoSchedModel, []>;
 572: def : CK807<"ck807f", NoSchedModel,
 573:             [FeatureFPUV2_SF, FeatureFPUV2_DF, FeatureFdivdu,
 574:              HasFLOATE1, HasFLOAT1E2, HasFLOAT1E3, HasFLOAT3E4]>;
 575: def : CK807<"c807f", NoSchedModel,
 576:             [FeatureFPUV2_SF, FeatureFPUV2_DF, FeatureFdivdu,
```

- EN: This range defines declarative TableGen records such as CK807, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 CK807 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 577-594

```tablegen
 577:              HasFLOATE1, HasFLOAT1E2, HasFLOAT1E3, HasFLOAT3E4]>;
 578: def : CK807<"r807f", NoSchedModel,
 579:             [FeatureFPUV2_SF, FeatureFPUV2_DF, FeatureFdivdu,
 580:              HasFLOATE1, HasFLOAT1E2, HasFLOAT1E3, HasFLOAT3E4]>;
 581: def : CK807<"ck807ef", NoSchedModel, [
 582:   FeatureFPUV2_SF, FeatureFPUV2_DF,
 583:   FeatureFdivdu, HasFLOATE1, HasFLOAT1E2, HasFLOAT1E3, HasFLOAT3E4]>;
 584: 
 585: // CK810 series
 586: class CK810<string n, SchedMachineModel m, list<SubtargetFeature> f,
 587:             list<SubtargetFeature> tunef = []>
 588:     : ProcessorModel<n, m, !listconcat(f,
 589:     [ Has7E10, HasMP, HasMP1E2, FeatureTrust, FeatureHWDiv, FeatureDSP, HasDSP1E2, HasDSPE60,
 590:       FeatureHighreg, FeatureHardTP, FeatureNVIC, FeatureCache, ProcCK810]), !listconcat(tunef, [])>;
 591: 
 592: def : CK810<"ck810", NoSchedModel, []>;
 593: def : CK810<"ck810e", NoSchedModel, []>;
 594: def : CK810<"ck810t", NoSchedModel, []>;
```

- EN: This range defines declarative TableGen records such as CK810, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 CK810 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 595-612

```tablegen
 595: def : CK810<"ck810et", NoSchedModel, []>;
 596: def : CK810<"c810", NoSchedModel,
 597:             [FeatureFPUV2_SF, FeatureFPUV2_DF, FeatureFdivdu,
 598:              HasFLOATE1, HasFLOAT1E2]>;
 599: def : CK810<"ck810f", NoSchedModel,
 600:             [FeatureFPUV2_SF, FeatureFPUV2_DF, FeatureFdivdu,
 601:              HasFLOATE1, HasFLOAT1E2]>;
 602: def : CK810<"ck810ef", NoSchedModel,
 603:             [FeatureFPUV2_SF, FeatureFPUV2_DF, FeatureFdivdu,
 604:              HasFLOATE1, HasFLOAT1E2]>;
 605: def : CK810<"ck810ft", NoSchedModel,
 606:             [FeatureFPUV2_SF, FeatureFPUV2_DF, FeatureFdivdu,
 607:              HasFLOATE1, HasFLOAT1E2]>;
 608: def : CK810<"ck810eft", NoSchedModel,
 609:             [FeatureFPUV2_SF, FeatureFPUV2_DF, FeatureFdivdu,
 610:              HasFLOATE1, HasFLOAT1E2]>;
 611: def : CK810<"c810t", NoSchedModel,
 612:             [FeatureFPUV2_SF, FeatureFPUV2_DF, FeatureFdivdu,
```

- EN: This range continues the implementation of the backend component described by CSKY.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 613-630

```tablegen
 613:              HasFLOATE1, HasFLOAT1E2]>;
 614: 
 615: class CK810V<string n, SchedMachineModel m, list<SubtargetFeature> f,
 616:              list<SubtargetFeature> tunef = []>
 617:     : CK810<n, m, !listconcat(f, [FeatureVDSPV1_128, ProcCK810V]), !listconcat(tunef, [])>;
 618: 
 619: def : CK810V<"ck810v", NoSchedModel, []>;
 620: def : CK810V<"ck810ev", NoSchedModel, []>;
 621: def : CK810V<"ck810tv", NoSchedModel, []>;
 622: def : CK810V<"ck810etv", NoSchedModel, []>;
 623: def : CK810V<"ck810fv", NoSchedModel, [
 624:   FeatureFPUV2_SF, FeatureFPUV2_DF, FeatureFdivdu,
 625:   HasFLOATE1, HasFLOAT1E2
 626: ]>;
 627: def : CK810V<"ck810efv", NoSchedModel, [
 628:   FeatureFPUV2_SF, FeatureFPUV2_DF, FeatureFdivdu,
 629:   HasFLOATE1, HasFLOAT1E2
 630: ]>;
```

- EN: This range defines declarative TableGen records such as CK810V, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 CK810V 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 631-648

```tablegen
 631: def : CK810V<"c810v", NoSchedModel, [
 632:   FeatureFPUV2_SF, FeatureFPUV2_DF, FeatureFdivdu,
 633:   HasFLOATE1, HasFLOAT1E2
 634: ]>;
 635: def : CK810V<"ck810ftv", NoSchedModel, [
 636:   FeatureFPUV2_SF, FeatureFPUV2_DF, FeatureFdivdu,
 637:   HasFLOATE1, HasFLOAT1E2
 638: ]>;
 639: def : CK810V<"ck810eftv", NoSchedModel, [
 640:   FeatureFPUV2_SF, FeatureFPUV2_DF, FeatureFdivdu,
 641:   HasFLOATE1, HasFLOAT1E2
 642: ]>;
 643: def : CK810V<"c810tv", NoSchedModel, [
 644:   FeatureFPUV2_SF, FeatureFPUV2_DF, FeatureFdivdu,
 645:   HasFLOATE1, HasFLOAT1E2
 646: ]>;
 647: 
 648: // CK860 series
```

- EN: This range continues the implementation of the backend component described by CSKY.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 649-666

```tablegen
 649: class CK860<string n, SchedMachineModel m, list<SubtargetFeature> f,
 650:             list<SubtargetFeature> tunef = []>
 651:     : ProcessorModel<n, m, !listconcat(f,
 652:     [ Has10E60, HasMP, HasMP1E2, Has3r1E3r2, Has3r2E3r3, FeatureTrust, FeatureBTST16, FeatureHWDiv, HasDSPE60,
 653:       FeatureHighreg, FeatureHardTP, FeatureNVIC, FeatureCache, ProcCK860]), !listconcat(tunef, [])>;
 654: 
 655: class CK860V<string n, SchedMachineModel m, list<SubtargetFeature> f,
 656:              list<SubtargetFeature> tunef = []>
 657:     : CK860<n, m, !listconcat(f, [FeatureVDSPV2, HasVDSP2E60F, ProcCK860V]), !listconcat(tunef, [])>;
 658: 
 659: def : CK860<"ck860", NoSchedModel, []>;
 660: def : CK860<"ck860f", NoSchedModel,
 661:             [FeatureFPUV3_HI, FeatureFPUV3_HF, FeatureFPUV3_SF, FeatureFPUV3_DF, HasFLOAT7E60]>;
 662: def : CK860<"c860", NoSchedModel,
 663:             [FeatureFPUV3_HI, FeatureFPUV3_HF, FeatureFPUV3_SF, FeatureFPUV3_DF, HasFLOAT7E60]>;
 664: def : CK860V<"c860v", NoSchedModel,
 665:              [FeatureFPUV3_HI, FeatureFPUV3_HF, FeatureFPUV3_SF, FeatureFPUV3_DF, HasFLOAT7E60]>;
 666: def : CK860V<"ck860v", NoSchedModel, []>;
```

- EN: This range defines declarative TableGen records such as CK860, CK860V, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 CK860, CK860V 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 667-684

```tablegen
 667: def : CK860V<"ck860fv", NoSchedModel,
 668:              [FeatureFPUV3_HI, FeatureFPUV3_HF, FeatureFPUV3_SF, FeatureFPUV3_DF, HasFLOAT7E60]>;
 669: 
 670: //===----------------------------------------------------------------------===//
 671: // Define the CSKY target.
 672: //===----------------------------------------------------------------------===//
 673: 
 674: defm : RemapAllTargetPseudoPointerOperands<GPR>;
 675: 
 676: def CSKYInstrInfo : InstrInfo;
 677: 
 678: 
 679: def CSKYAsmParser : AsmParser {
 680:   let ShouldEmitMatchRegisterAltName = 1;
 681:   let AllowDuplicateRegisterNames = 1;
 682: }
 683: 
 684: def CSKYAsmWriter : AsmWriter {
```

- EN: This range defines declarative TableGen records such as CSKYInstrInfo, CSKYAsmParser, CSKYAsmWriter, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CSKYInstrInfo, CSKYAsmParser, CSKYAsmWriter 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 685-693

```tablegen
 685:   int PassSubtarget = 1;
 686: }
 687: 
 688: def CSKY : Target {
 689:   let InstructionSet = CSKYInstrInfo;
 690:   let AssemblyParsers = [CSKYAsmParser];
 691:   let AssemblyWriters = [CSKYAsmWriter];
 692:   let AllowRegisterRenaming = 1;
 693: }
```

- EN: This range defines declarative TableGen records such as CSKY, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CSKY 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- Target-wide metadata / 全局目标元数据
- CPU feature modelling / CPU 特性建模
- Assembly parsing / 汇编解析
- Pseudo-instruction handling / 伪指令处理
- Scheduling model / 调度模型
- Calling convention / 调用约定

## Dependencies / 依赖关系

- Direct includes / 直接包含: `llvm/Target/Target.td`, `CSKYRegisterInfo.td`, `CSKYCallingConv.td`, `CSKYInstrInfo.td`
- LLVM subsystems / LLVM 子系统: TableGen
- Local companions / 本地配套文件: `CSKY.h`
