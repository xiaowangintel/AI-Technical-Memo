# LoongArch.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArch.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative LoongArch backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 LoongArch 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```tablegen
   1: //===-- LoongArch.td - Describe the LoongArch Target -------*- tablegen -*-===//
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
  12: // LoongArch subtarget features and instruction predicates.
  13: //===----------------------------------------------------------------------===//
  14: 
  15: // LoongArch is divided into two versions, the 32-bit version (LA32) and the
  16: // 64-bit version (LA64).
  17: 
  18: // LoongArch 32-bit is divided into two variants, the reduced 32-bit variant
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It composes TableGen records by including `Target.td` and reusing previously declared backend fragments.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它通过包含 `Target.td` 组合 TableGen 记录，并复用此前声明的后端片段。

### Lines 19-36 / 第 19-36 行
```tablegen
  19: // (LA32R) and the standard 32-bit variant (LA32S).
  20: def Feature32S
  21:     : SubtargetFeature<"32s", "Has32S", "true",
  22:                        "LA32 Standard Basic Instruction Extension">;
  23: def Has32S : Predicate<"Subtarget->has32S()">;
  24: 
  25: def Feature64Bit
  26:     : SubtargetFeature<"64bit", "HasLA64", "true",
  27:                        "LA64 Basic Integer and Privilege Instruction Set",
  28:                        [Feature32S]>;
  29: def Feature32Bit
  30:     : SubtargetFeature<"32bit", "HasLA32", "true",
  31:                        "LA32 Basic Integer and Privilege Instruction Set">;
  32: def IsLA64
  33:     : Predicate<"Subtarget->is64Bit()">,
  34:       AssemblerPredicate<(all_of Feature64Bit),
  35:                          "LA64 Basic Integer and Privilege Instruction Set">;
  36: def IsLA32
```
- **EN**: This block declares or refines TableGen records such as `Feature32S`, `Has32S`, `Feature64Bit`, `Feature32Bit`, `IsLA64`, `IsLA32`.
- **CN**: 该代码块声明或细化了 `Feature32S`, `Has32S`, `Feature64Bit`, `Feature32Bit`, `IsLA64`, `IsLA32` 等 TableGen 记录。

### Lines 37-54 / 第 37-54 行
```tablegen
  37:     : Predicate<"!Subtarget->is64Bit()">,
  38:       AssemblerPredicate<(all_of(not Feature64Bit)),
  39:                          "LA32 Basic Integer and Privilege Instruction Set">;
  40: 
  41: defvar LA32 = DefaultMode;
  42: def LA64 : HwMode<[IsLA64]>;
  43: 
  44: // Single Precision floating point
  45: def FeatureBasicF
  46:     : SubtargetFeature<"f", "HasBasicF", "true",
  47:                        "'F' (Single-Precision Floating-Point)">;
  48: def HasBasicF : Predicate<"Subtarget->hasBasicF()">;
  49: 
  50: // Double Precision floating point
  51: def FeatureBasicD
  52:     : SubtargetFeature<"d", "HasBasicD", "true",
  53:                        "'D' (Double-Precision Floating-Point)",
  54:                        [FeatureBasicF]>;
```
- **EN**: This block declares or refines TableGen records such as `LA64`, `FeatureBasicF`, `HasBasicF`, `FeatureBasicD`.
- **CN**: 该代码块声明或细化了 `LA64`, `FeatureBasicF`, `HasBasicF`, `FeatureBasicD` 等 TableGen 记录。

### Lines 55-72 / 第 55-72 行
```tablegen
  55: def HasBasicD : Predicate<"Subtarget->hasBasicD()">;
  56: 
  57: // Loongson SIMD eXtension (LSX)
  58: def FeatureExtLSX
  59:     : SubtargetFeature<"lsx", "HasExtLSX", "true",
  60:                        "'LSX' (Loongson SIMD Extension)", [FeatureBasicD]>;
  61: def HasExtLSX : Predicate<"Subtarget->hasExtLSX()">;
  62: 
  63: // Loongson Advanced SIMD eXtension (LASX)
  64: def FeatureExtLASX
  65:     : SubtargetFeature<"lasx", "HasExtLASX", "true",
  66:                        "'LASX' (Loongson Advanced SIMD Extension)",
  67:                        [FeatureExtLSX]>;
  68: def HasExtLASX : Predicate<"Subtarget->hasExtLASX()">;
  69: 
  70: // Loongson VirtualiZation (LVZ)
  71: def FeatureExtLVZ
  72:     : SubtargetFeature<"lvz", "HasExtLVZ", "true",
```
- **EN**: This block declares or refines TableGen records such as `HasBasicD`, `FeatureExtLSX`, `HasExtLSX`, `FeatureExtLASX`, `HasExtLASX`, `FeatureExtLVZ`.
- **CN**: 该代码块声明或细化了 `HasBasicD`, `FeatureExtLSX`, `HasExtLSX`, `FeatureExtLASX`, `HasExtLASX`, `FeatureExtLVZ` 等 TableGen 记录。

### Lines 73-90 / 第 73-90 行
```tablegen
  73:                        "'LVZ' (Loongson Virtualization Extension)">;
  74: def HasExtLVZ : Predicate<"Subtarget->hasExtLVZ()">;
  75: 
  76: // Loongson Binary Translation (LBT)
  77: def FeatureExtLBT
  78:     : SubtargetFeature<"lbt", "HasExtLBT", "true",
  79:                        "'LBT' (Loongson Binary Translation Extension)">;
  80: def HasExtLBT : Predicate<"Subtarget->hasExtLBT()">;
  81: 
  82: // Expand la.global as la.pcrel
  83: def LaGlobalWithPcrel
  84:     : SubtargetFeature<"la-global-with-pcrel", "HasLaGlobalWithPcrel", "true",
  85:                        "Expand la.global as la.pcrel">;
  86: def HasLaGlobalWithPcrel
  87:     : Predicate<"Subtarget->hasLaGlobalWithPcrel()">,
  88:       AssemblerPredicate<(all_of LaGlobalWithPcrel),
  89:                          "Expand la.global as la.pcrel">;
  90: 
```
- **EN**: This block declares or refines TableGen records such as `HasExtLVZ`, `FeatureExtLBT`, `HasExtLBT`, `LaGlobalWithPcrel`, `HasLaGlobalWithPcrel`.
- **CN**: 该代码块声明或细化了 `HasExtLVZ`, `FeatureExtLBT`, `HasExtLBT`, `LaGlobalWithPcrel`, `HasLaGlobalWithPcrel` 等 TableGen 记录。

### Lines 91-108 / 第 91-108 行
```tablegen
  91: // Expand la.global as la.abs
  92: def LaGlobalWithAbs
  93:     : SubtargetFeature<"la-global-with-abs", "HasLaGlobalWithAbs", "true",
  94:                        "Expand la.global as la.abs">;
  95: def HasLaGlobalWithAbs
  96:     : Predicate<"Subtarget->hasLaGlobalWithAbs()">,
  97:       AssemblerPredicate<(all_of LaGlobalWithAbs),
  98:                          "Expand la.global as la.abs">;
  99: 
 100: // Expand la.local as la.abs
 101: def LaLocalWithAbs
 102:     : SubtargetFeature<"la-local-with-abs", "HasLaLocalWithAbs", "true",
 103:                        "Expand la.local as la.abs">;
 104: def HasLaLocalWithAbs
 105:     : Predicate<"Subtarget->hasLaLocalWithAbs()">,
 106:       AssemblerPredicate<(all_of LaLocalWithAbs),
 107:                          "Expand la.local as la.abs">;
 108: 
```
- **EN**: This block declares or refines TableGen records such as `LaGlobalWithAbs`, `HasLaGlobalWithAbs`, `LaLocalWithAbs`, `HasLaLocalWithAbs`.
- **CN**: 该代码块声明或细化了 `LaGlobalWithAbs`, `HasLaGlobalWithAbs`, `LaLocalWithAbs`, `HasLaLocalWithAbs` 等 TableGen 记录。

### Lines 109-126 / 第 109-126 行
```tablegen
 109: // Unaligned memory access
 110: def FeatureUAL
 111:     : SubtargetFeature<"ual", "HasUAL", "true",
 112:                        "Allow memory accesses to be unaligned">;
 113: 
 114: def FeatureRelax
 115:     : SubtargetFeature<"relax", "HasLinkerRelax", "true",
 116:                        "Enable Linker relaxation">;
 117: 
 118: // Floating point approximation operation
 119: def FeatureFrecipe
 120:     : SubtargetFeature<"frecipe", "HasFrecipe", "true",
 121:                        "Support frecipe.{s/d} and frsqrte.{s/d} instructions">;
 122: def HasFrecipe : Predicate<"Subtarget->hasFrecipe()">;
 123: 
 124: // Atomic memory swap and add instructions for byte and half word
 125: def FeatureLAM_BH
 126:     : SubtargetFeature<"lam-bh", "HasLAM_BH", "true",
```
- **EN**: This block declares or refines TableGen records such as `FeatureUAL`, `FeatureRelax`, `FeatureFrecipe`, `HasFrecipe`, `FeatureLAM_BH`.
- **CN**: 该代码块声明或细化了 `FeatureUAL`, `FeatureRelax`, `FeatureFrecipe`, `HasFrecipe`, `FeatureLAM_BH` 等 TableGen 记录。

### Lines 127-144 / 第 127-144 行
```tablegen
 127:                         "Support amswap[_db].{b/h} and amadd[_db].{b/h} instructions">;
 128: def HasLAM_BH : Predicate<"Subtarget->hasLAM_BH()">;
 129: 
 130: // Atomic memory compare and swap instructions for byte, half word, word and double word
 131: def FeatureLAMCAS
 132:     : SubtargetFeature<"lamcas", "HasLAMCAS", "true",
 133:                         "Support amcas[_db].{b/h/w/d}">;
 134: def HasLAMCAS : Predicate<"Subtarget->hasLAMCAS()">;
 135: 
 136: def FeatureLD_SEQ_SA
 137:     : SubtargetFeature<"ld-seq-sa", "HasLD_SEQ_SA", "true",
 138:                         "Don't use a same-address load-load barrier (dbar 0x700)">;
 139: def HasLD_SEQ_SA : Predicate<"Subtarget->hasLD_SEQ_SA()">;
 140: 
 141: // Assume div.w[u] and mod.w[u] can handle inputs that are not sign-extended.
 142: def FeatureDiv32
 143:     : SubtargetFeature<"div32", "HasDiv32", "true",
 144:                         "Assume div.w[u] and mod.w[u] can handle inputs that are not sign-extended">;
```
- **EN**: This block declares or refines TableGen records such as `HasLAM_BH`, `FeatureLAMCAS`, `HasLAMCAS`, `FeatureLD_SEQ_SA`, `HasLD_SEQ_SA`, `FeatureDiv32`.
- **CN**: 该代码块声明或细化了 `HasLAM_BH`, `FeatureLAMCAS`, `HasLAMCAS`, `FeatureLD_SEQ_SA`, `HasLD_SEQ_SA`, `FeatureDiv32` 等 TableGen 记录。

### Lines 145-162 / 第 145-162 行
```tablegen
 145: def HasDiv32 : Predicate<"Subtarget->hasDiv32()">;
 146: 
 147: // Support SC.Q instruction
 148: def FeatureSCQ
 149:     : SubtargetFeature<"scq", "HasSCQ", "true",
 150:                         "Support sc.q instruction">;
 151: def HasSCQ : Predicate<"Subtarget->hasSCQ()">;
 152: 
 153: def TunePreferWInst
 154:     : SubtargetFeature<"prefer-w-inst", "PreferWInst", "true",
 155:                        "Prefer instructions with W suffix">;
 156: 
 157: //===----------------------------------------------------------------------===//
 158: // Registers, instruction descriptions ...
 159: //===----------------------------------------------------------------------===//
 160: 
 161: include "LoongArchRegisterInfo.td"
 162: include "LoongArchCallingConv.td"
```
- **EN**: It composes TableGen records by including `LoongArchRegisterInfo.td`, `LoongArchCallingConv.td` and reusing previously declared backend fragments. This block declares or refines TableGen records such as `HasDiv32`, `FeatureSCQ`, `HasSCQ`, `TunePreferWInst`.
- **CN**: 它通过包含 `LoongArchRegisterInfo.td`, `LoongArchCallingConv.td` 组合 TableGen 记录，并复用此前声明的后端片段。 该代码块声明或细化了 `HasDiv32`, `FeatureSCQ`, `HasSCQ`, `TunePreferWInst` 等 TableGen 记录。

### Lines 163-180 / 第 163-180 行
```tablegen
 163: include "LoongArchInstrInfo.td"
 164: 
 165: //===----------------------------------------------------------------------===//
 166: // LoongArch processors supported.
 167: //===----------------------------------------------------------------------===//
 168: 
 169: def : ProcessorModel<"generic-la32", NoSchedModel, [Feature32Bit]>;
 170: def : ProcessorModel<"generic-la64", NoSchedModel, [Feature64Bit,
 171:                                                     FeatureUAL,
 172:                                                     FeatureExtLSX]>;
 173: 
 174: // Generic 32-bit processor.
 175: def : ProcessorModel<"loongarch32", NoSchedModel, [Feature32Bit]>;
 176: 
 177: // Generic 64-bit processor with double-precision floating-point support.
 178: def : ProcessorModel<"loongarch64", NoSchedModel, [Feature64Bit,
 179:                                                    FeatureUAL,
 180:                                                    FeatureBasicD]>;
```
- **EN**: It composes TableGen records by including `LoongArchInstrInfo.td` and reusing previously declared backend fragments.
- **CN**: 它通过包含 `LoongArchInstrInfo.td` 组合 TableGen 记录，并复用此前声明的后端片段。

### Lines 181-198 / 第 181-198 行
```tablegen
 181: 
 182: // Support generic for compatibility with other targets. The triple will be used
 183: // to change to the appropriate la32/la64 version.
 184: def : ProcessorModel<"generic", NoSchedModel, []>;
 185: 
 186: def : ProcessorModel<"la464", NoSchedModel, [Feature64Bit,
 187:                                              FeatureUAL,
 188:                                              FeatureExtLASX,
 189:                                              FeatureExtLVZ,
 190:                                              FeatureExtLBT]>;
 191: 
 192: def : ProcessorModel<"la664", NoSchedModel, [Feature64Bit,
 193:                                              FeatureUAL,
 194:                                              FeatureExtLASX,
 195:                                              FeatureExtLVZ,
 196:                                              FeatureExtLBT,
 197:                                              FeatureFrecipe,
 198:                                              FeatureLAM_BH,
```
- **EN**: This span continues the file's main responsibility: defines declarative LoongArch backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 199-216 / 第 199-216 行
```tablegen
 199:                                              FeatureLAMCAS,
 200:                                              FeatureLD_SEQ_SA,
 201:                                              FeatureDiv32,
 202:                                              FeatureSCQ]>;
 203: 
 204: //===----------------------------------------------------------------------===//
 205: // Define the LoongArch target.
 206: //===----------------------------------------------------------------------===//
 207: 
 208: defm : RemapAllTargetPseudoPointerOperands<GPR>;
 209: 
 210: def LoongArchInstrInfo : InstrInfo {
 211:   let guessInstructionProperties = 0;
 212: }
 213: 
 214: def LoongArchAsmParser : AsmParser {
 215:   let ShouldEmitMatchRegisterAltName = 1;
 216:   let AllowDuplicateRegisterNames = 1;
```
- **EN**: This block declares or refines TableGen records such as `LoongArchInstrInfo`, `LoongArchAsmParser`.
- **CN**: 该代码块声明或细化了 `LoongArchInstrInfo`, `LoongArchAsmParser` 等 TableGen 记录。

### Lines 217-234 / 第 217-234 行
```tablegen
 217: }
 218: 
 219: def LoongArchAsmParserVariant : AsmParserVariant {
 220:   int Variant = 0;
 221:   // Recognize hard coded registers.
 222:   string RegisterPrefix = "$";
 223: }
 224: 
 225: def LoongArchAsmWriter : AsmWriter {
 226:   int PassSubtarget = 1;
 227: }
 228: 
 229: def LoongArch : Target {
 230:   let InstructionSet = LoongArchInstrInfo;
 231:   let AssemblyParsers = [LoongArchAsmParser];
 232:   let AssemblyParserVariants = [LoongArchAsmParserVariant];
 233:   let AssemblyWriters = [LoongArchAsmWriter];
 234:   let AllowRegisterRenaming = 1;
```
- **EN**: This block declares or refines TableGen records such as `LoongArchAsmParserVariant`, `LoongArchAsmWriter`, `LoongArch`.
- **CN**: 该代码块声明或细化了 `LoongArchAsmParserVariant`, `LoongArchAsmWriter`, `LoongArch` 等 TableGen 记录。

### Lines 235-235 / 第 235-235 行
```tablegen
 235: }
```
- **EN**: This span continues the file's main responsibility: defines declarative LoongArch backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **Assembly parsing**: Reads textual assembly operands, directives, and mnemonics. / 读取文本汇编的操作数、伪指令和助记符。
- **Calling convention**: Defines how arguments, returns, and preserved registers are assigned. / 定义参数、返回值和保留寄存器的分配方式。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `llvm/Target/Target.td`
- `LoongArchRegisterInfo.td`
- `LoongArchCallingConv.td`
- `LoongArchInstrInfo.td`
