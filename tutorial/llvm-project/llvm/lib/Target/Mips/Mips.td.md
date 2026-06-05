# Mips.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/Mips.td`
- Repository: `llvm-project`
- Purpose (EN): Defines `Mips` in LLVM TableGen DSL for the Mips backend, covering target descriptions, instruction records, and code-generation metadata.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `Mips`，涵盖目标描述、指令记录以及代码生成元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-9
```tablegen
//===-- Mips.td - Describe the Mips Target Machine ---------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
// This is the top level entry point for the Mips target.
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 11-13
```tablegen
//===----------------------------------------------------------------------===//
// Target-independent interfaces
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 15-15
```tablegen
include "llvm/Target/Target.td"
```
- EN: Pulls in other TableGen fragments so this file can reuse common records, predicates, and target-wide definitions.
- CN: 这里引入其他 TableGen 片段，以便复用公共记录、谓词以及整个目标范围内的定义。

### Lines 17-34
```tablegen
// The overall idea of the PredicateControl class is to chop the Predicates list
// into subsets that are usually overridden independently. This allows
// subclasses to partially override the predicates of their superclasses without
// having to re-add all the existing predicates.
class PredicateControl {
  // Predicates for the encoding scheme in use such as HasStdEnc
  list<Predicate> EncodingPredicates = [];
  // Predicates for the GPR size such as IsGP64bit
  list<Predicate> GPRPredicates = [];
  // Predicates for the PTR size such as IsPTR64bit
  list<Predicate> PTRPredicates = [];
  // Predicates for a symbol's size such as hasSym32.
  list<Predicate> SYMPredicates = [];
  // Predicates for the FGR size and layout such as IsFP64bit
  list<Predicate> FGRPredicates = [];
  // Predicates for the instruction group membership such as ISA's.
  list<Predicate> InsnPredicates = [];
  // Predicate for the ASE that an instruction belongs to.
```
- EN: Declares reusable TableGen class `is` for `Mips`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips` 声明可复用的 TableGen 类 `is`，通常用于抽象共享字段、谓词或编码结构。

### Lines 35-49
```tablegen
  list<Predicate> ASEPredicate = [];
  // Predicate for marking the instruction as usable in hard-float mode only.
  list<Predicate> HardFloatPredicate = [];
  // Predicates for anything else
  list<Predicate> AdditionalPredicates = [];
  list<Predicate> Predicates = !listconcat(EncodingPredicates,
                                           GPRPredicates,
                                           PTRPredicates,
                                           SYMPredicates,
                                           FGRPredicates,
                                           InsnPredicates,
                                           HardFloatPredicate,
                                           ASEPredicate,
                                           AdditionalPredicates);
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 51-54
```tablegen
// Like Requires<> but for the AdditionalPredicates list
class AdditionalRequires<list<Predicate> preds> {
  list<Predicate> AdditionalPredicates = preds;
}
```
- EN: Declares reusable TableGen class `AdditionalRequires` for `Mips`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips` 声明可复用的 TableGen 类 `AdditionalRequires`，通常用于抽象共享字段、谓词或编码结构。

### Lines 56-58
```tablegen
//===----------------------------------------------------------------------===//
// Mips Subtarget features                                                    //
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 60-77
```tablegen
def FeatureNoABICalls  : SubtargetFeature<"noabicalls", "NoABICalls", "true",
                                "Disable SVR4-style position-independent code">;
def FeaturePTR64Bit    : SubtargetFeature<"ptr64", "IsPTR64bit", "true",
                                "Pointers are 64-bit wide">;
def FeatureGP64Bit     : SubtargetFeature<"gp64", "IsGP64bit", "true",
                                "General Purpose Registers are 64-bit wide">;
def FeatureFP64Bit     : SubtargetFeature<"fp64", "IsFP64bit", "true",
                                "Support 64-bit FP registers">;
def FeatureFPXX        : SubtargetFeature<"fpxx", "IsFPXX", "true",
                                "Support for FPXX">;
def FeatureNaN2008     : SubtargetFeature<"nan2008", "IsNaN2008bit", "true",
                                "IEEE 754-2008 NaN encoding">;
def FeatureAbs2008     : SubtargetFeature<"abs2008", "Abs2008", "true",
                                          "Disable IEEE 754-2008 abs.fmt mode">;
def FeatureSingleFloat : SubtargetFeature<"single-float", "IsSingleFloat",
                                "true", "Only supports single precision float">;
def FeatureSoftFloat   : SubtargetFeature<"soft-float", "IsSoftFloat", "true",
                                "Does not support floating point instructions">;
```
- EN: Defines TableGen record `FeatureNoABICalls` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureNoABICalls`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 78-95
```tablegen
def FeatureNoOddSPReg  : SubtargetFeature<"nooddspreg", "UseOddSPReg", "false",
                              "Disable odd numbered single-precision "
                              "registers">;
def FeatureVFPU        : SubtargetFeature<"vfpu", "HasVFPU",
                                "true", "Enable vector FPU instructions">;
def FeatureMips1       : SubtargetFeature<"mips1", "MipsArchVersion", "Mips1",
                                "Mips I ISA Support [highly experimental]">;
def FeatureMips2       : SubtargetFeature<"mips2", "MipsArchVersion", "Mips2",
                                "Mips II ISA Support [highly experimental]",
                                [FeatureMips1]>;
def FeatureMips3_32    : SubtargetFeature<"mips3_32", "HasMips3_32", "true",
                                "Subset of MIPS-III that is also in MIPS32 "
                                "[highly experimental]">;
def FeatureMips3_32r2  : SubtargetFeature<"mips3_32r2", "HasMips3_32r2", "true",
                                "Subset of MIPS-III that is also in MIPS32r2 "
                                "[highly experimental]">;
def FeatureMips3       : SubtargetFeature<"mips3", "MipsArchVersion", "Mips3",
                                "MIPS III ISA Support [highly experimental]",
```
- EN: Defines TableGen record `FeatureNoOddSPReg` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureNoOddSPReg`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 96-113
```tablegen
                                [FeatureMips2, FeatureMips3_32,
                                 FeatureMips3_32r2, FeatureGP64Bit,
                                 FeatureFP64Bit]>;
def FeatureMips4_32    : SubtargetFeature<"mips4_32", "HasMips4_32", "true",
                                "Subset of MIPS-IV that is also in MIPS32 "
                                "[highly experimental]">;
def FeatureMips4_32r2  : SubtargetFeature<"mips4_32r2", "HasMips4_32r2", "true",
                                "Subset of MIPS-IV that is also in MIPS32r2 "
                                "[highly experimental]">;
def FeatureMips4       : SubtargetFeature<"mips4", "MipsArchVersion",
                                "Mips4", "MIPS IV ISA Support",
                                [FeatureMips3, FeatureMips4_32,
                                 FeatureMips4_32r2]>;
def FeatureMips5_32r2  : SubtargetFeature<"mips5_32r2", "HasMips5_32r2", "true",
                                "Subset of MIPS-V that is also in MIPS32r2 "
                                "[highly experimental]">;
def FeatureMips5       : SubtargetFeature<"mips5", "MipsArchVersion", "Mips5",
                                "MIPS V ISA Support [highly experimental]",
```
- EN: Defines TableGen record `FeatureMips4_32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureMips4_32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 114-131
```tablegen
                                [FeatureMips4, FeatureMips5_32r2]>;
def FeatureMips32      : SubtargetFeature<"mips32", "MipsArchVersion", "Mips32",
                                "Mips32 ISA Support",
                                [FeatureMips2, FeatureMips3_32,
                                 FeatureMips4_32]>;
def FeatureMips32r2    : SubtargetFeature<"mips32r2", "MipsArchVersion",
                                "Mips32r2", "Mips32r2 ISA Support",
                                [FeatureMips3_32r2, FeatureMips4_32r2,
                                 FeatureMips5_32r2, FeatureMips32]>;
def FeatureMips32r3    : SubtargetFeature<"mips32r3", "MipsArchVersion",
                                "Mips32r3", "Mips32r3 ISA Support",
                                [FeatureMips32r2]>;
def FeatureMips32r5    : SubtargetFeature<"mips32r5", "MipsArchVersion",
                                "Mips32r5", "Mips32r5 ISA Support",
                                [FeatureMips32r3]>;
def FeatureMips32r6    : SubtargetFeature<"mips32r6", "MipsArchVersion",
                                "Mips32r6",
                                "Mips32r6 ISA Support [experimental]",
```
- EN: Defines TableGen record `FeatureMips32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureMips32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 132-149
```tablegen
                                [FeatureMips32r5, FeatureFP64Bit,
                                 FeatureNaN2008, FeatureAbs2008]>;
def FeatureMips64      : SubtargetFeature<"mips64", "MipsArchVersion",
                                "Mips64", "Mips64 ISA Support",
                                [FeatureMips5, FeatureMips32]>;
def FeatureMips64r2    : SubtargetFeature<"mips64r2", "MipsArchVersion",
                                "Mips64r2", "Mips64r2 ISA Support",
                                [FeatureMips64, FeatureMips32r2]>;
def FeatureMips64r3    : SubtargetFeature<"mips64r3", "MipsArchVersion",
                                "Mips64r3", "Mips64r3 ISA Support",
                                [FeatureMips64r2, FeatureMips32r3]>;
def FeatureMips64r5    : SubtargetFeature<"mips64r5", "MipsArchVersion",
                                "Mips64r5", "Mips64r5 ISA Support",
                                [FeatureMips64r3, FeatureMips32r5]>;
def FeatureMips64r6    : SubtargetFeature<"mips64r6", "MipsArchVersion",
                                "Mips64r6",
                                "Mips64r6 ISA Support [experimental]",
                                [FeatureMips32r6, FeatureMips64r5,
```
- EN: Defines TableGen record `FeatureMips64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureMips64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 150-152
```tablegen
                                 FeatureNaN2008, FeatureAbs2008]>;
def FeatureSym32       : SubtargetFeature<"sym32", "HasSym32", "true",
                                          "Symbols are 32 bit on Mips64">;
```
- EN: Defines TableGen record `FeatureSym32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureSym32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 154-155
```tablegen
def FeatureMips16  : SubtargetFeature<"mips16", "InMips16Mode", "true",
                                      "Mips16 mode">;
```
- EN: Defines TableGen record `FeatureMips16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureMips16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 157-162
```tablegen
def FeatureDSP : SubtargetFeature<"dsp", "HasDSP", "true", "Mips DSP ASE">;
def FeatureDSPR2 : SubtargetFeature<"dspr2", "HasDSPR2", "true",
                                    "Mips DSP-R2 ASE", [FeatureDSP]>;
def FeatureDSPR3
    : SubtargetFeature<"dspr3", "HasDSPR3", "true", "Mips DSP-R3 ASE",
                       [ FeatureDSP, FeatureDSPR2 ]>;
```
- EN: Defines TableGen record `FeatureDSP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureDSP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 164-164
```tablegen
def FeatureMips3D : SubtargetFeature<"mips3d", "Has3D", "true", "Mips 3D ASE">;
```
- EN: Defines TableGen record `FeatureMips3D` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureMips3D`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 166-166
```tablegen
def FeatureMSA : SubtargetFeature<"msa", "HasMSA", "true", "Mips MSA ASE">;
```
- EN: Defines TableGen record `FeatureMSA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureMSA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 168-168
```tablegen
def FeatureEVA : SubtargetFeature<"eva", "HasEVA", "true", "Mips EVA ASE">;
```
- EN: Defines TableGen record `FeatureEVA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureEVA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 170-170
```tablegen
def FeatureCRC : SubtargetFeature<"crc", "HasCRC", "true", "Mips R6 CRC ASE">;
```
- EN: Defines TableGen record `FeatureCRC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureCRC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 172-173
```tablegen
def FeatureVirt : SubtargetFeature<"virt", "HasVirt", "true",
                                   "Mips Virtualization ASE">;
```
- EN: Defines TableGen record `FeatureVirt` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureVirt`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 175-176
```tablegen
def FeatureGINV : SubtargetFeature<"ginv", "HasGINV", "true",
                                   "Mips Global Invalidate ASE">;
```
- EN: Defines TableGen record `FeatureGINV` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureGINV`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 178-179
```tablegen
def FeatureMicroMips  : SubtargetFeature<"micromips", "InMicroMipsMode", "true",
                                         "microMips mode">;
```
- EN: Defines TableGen record `FeatureMicroMips` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureMicroMips`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 181-183
```tablegen
def FeatureCnMips : SubtargetFeature<"cnmips", "HasCnMips",
                                     "true", "Octeon cnMIPS Support",
                                     [FeatureMips64r2]>;
```
- EN: Defines TableGen record `FeatureCnMips` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureCnMips`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 185-187
```tablegen
def FeatureCnMipsP : SubtargetFeature<"cnmipsp", "HasCnMipsP",
                                      "true", "Octeon+ cnMIPS Support",
                                      [FeatureCnMips]>;
```
- EN: Defines TableGen record `FeatureCnMipsP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureCnMipsP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 189-190
```tablegen
def FeatureFixR5900 : SubtargetFeature<"fix-r5900", "FixR5900", "true",
                                       "Enable R5900 short loop erratum fix">;
```
- EN: Defines TableGen record `FeatureFixR5900` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureFixR5900`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 192-195
```tablegen
def FeatureR5900 : SubtargetFeature<"r5900", "IsR5900", "true",
                                    "R5900 (PS2 Emotion Engine) Support",
                                    [FeatureMips3, FeatureSingleFloat,
                                     FeatureFixR5900]>;
```
- EN: Defines TableGen record `FeatureR5900` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureR5900`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 197-200
```tablegen
def FeatureUseTCCInDIV : SubtargetFeature<
                               "use-tcc-in-div",
                               "UseTCCInDIV", "false",
                               "Force the assembler to use trapping">;
```
- EN: Defines TableGen record `FeatureUseTCCInDIV` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureUseTCCInDIV`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 202-204
```tablegen
def FeatureNoMadd4
    : SubtargetFeature<"nomadd4", "DisableMadd4", "true",
                       "Disable 4-operand madd.fmt and related instructions">;
```
- EN: Defines TableGen record `FeatureNoMadd4` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureNoMadd4`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 206-206
```tablegen
def FeatureMT : SubtargetFeature<"mt", "HasMT", "true", "Mips MT ASE">;
```
- EN: Defines TableGen record `FeatureMT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureMT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 208-209
```tablegen
def FeatureLongCalls : SubtargetFeature<"long-calls", "UseLongCalls", "true",
                                        "Disable use of the jal instruction">;
```
- EN: Defines TableGen record `FeatureLongCalls` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureLongCalls`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 211-212
```tablegen
def FeatureXGOT
    : SubtargetFeature<"xgot", "UseXGOT", "true", "Assume 32-bit GOT">;
```
- EN: Defines TableGen record `FeatureXGOT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureXGOT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 214-217
```tablegen
def FeatureUseIndirectJumpsHazard : SubtargetFeature<"use-indirect-jump-hazard",
                                                    "UseIndirectJumpsHazard",
                                                    "true", "Use indirect jump"
                        " guards to prevent certain speculation based attacks">;
```
- EN: Defines TableGen record `FeatureUseIndirectJumpsHazard` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureUseIndirectJumpsHazard`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 219-221
```tablegen
def FeatureStrictAlign
    : SubtargetFeature<"strict-align", "StrictAlign", "true",
                       "Disable unaligned load store for r6">;
```
- EN: Defines TableGen record `FeatureStrictAlign` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureStrictAlign`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 223-225
```tablegen
def FeatureUseCompactBranches
    : SubtargetFeature<"use-compact-branches", "UseCompactBranches", "true",
                       "Use compact branch instructions for MIPS32R6/MIPS64R6">;
```
- EN: Defines TableGen record `FeatureUseCompactBranches` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureUseCompactBranches`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 227-229
```tablegen
//===----------------------------------------------------------------------===//
// Mips Instruction Predicate Definitions.
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 231-236
```tablegen
def IsPTR64bit    :   Predicate<"Subtarget->isABI_N64()">,
                      AssemblerPredicate<(all_of FeaturePTR64Bit)>;
def IsPTR32bit    :   Predicate<"!Subtarget->isABI_N64()">,
                      AssemblerPredicate<(all_of (not FeaturePTR64Bit))>;
def UseCompactBranches : Predicate<"Subtarget->useCompactBranches()">,
                         AssemblerPredicate<(all_of FeatureUseCompactBranches)>;
```
- EN: Defines TableGen record `IsPTR64bit` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IsPTR64bit`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 238-240
```tablegen
//===----------------------------------------------------------------------===//
// HwModes
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 242-243
```tablegen
defvar MIPS32 = DefaultMode;
def MIPS64 : HwMode<[IsPTR64bit]>;
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 245-247
```tablegen
//===----------------------------------------------------------------------===//
// Register File, Calling Conv, Instruction Descriptions
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 249-253
```tablegen
include "MipsRegisterInfo.td"
include "MipsInstrInfo.td"
include "MipsCallingConv.td"
include "MipsRegisterBanks.td"
include "MipsCombine.td"
```
- EN: Pulls in other TableGen fragments so this file can reuse common records, predicates, and target-wide definitions.
- CN: 这里引入其他 TableGen 片段，以便复用公共记录、谓词以及整个目标范围内的定义。

### Lines 255-258
```tablegen
// Avoid forward declaration issues.
include "MipsScheduleI6400.td"
include "MipsScheduleP5600.td"
include "MipsScheduleGeneric.td"
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 260-260
```tablegen
defm : RemapAllTargetPseudoPointerOperands<mips_ptr_rc>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 262-263
```tablegen
def MipsInstrInfo : InstrInfo {
}
```
- EN: Defines TableGen record `MipsInstrInfo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MipsInstrInfo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 265-267
```tablegen
//===----------------------------------------------------------------------===//
// Mips processors supported.
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 269-271
```tablegen
def ImplP5600 : SubtargetFeature<"p5600", "ProcImpl",
                                 "MipsSubtarget::CPU::P5600",
                                 "The P5600 Processor", [FeatureMips32r5]>;
```
- EN: Defines TableGen record `ImplP5600` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ImplP5600`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 273-277
```tablegen
// The I6500 is the multi-cluster version of the I6400. Both are based on the
// same CPU architecture.
def ImplI6400
    : SubtargetFeature<"i6400", "ProcImpl", "MipsSubtarget::CPU::I6400",
                       "MIPS I6400 Processor", [FeatureMips64r6, FeatureMSA]>;
```
- EN: Defines TableGen record `ImplI6400` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ImplI6400`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 279-281
```tablegen
def ImplI6500
    : SubtargetFeature<"i6500", "ProcImpl", "MipsSubtarget::CPU::I6500",
                       "MIPS I6500 Processor", [FeatureMips64r6, FeatureMSA]>;
```
- EN: Defines TableGen record `ImplI6500` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ImplI6500`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 283-284
```tablegen
class Proc<string Name, list<SubtargetFeature> Features>
 : ProcessorModel<Name, MipsGenericModel, Features>;
```
- EN: Declares reusable TableGen class `Proc` for `Mips`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips` 声明可复用的 TableGen 类 `Proc`，通常用于抽象共享字段、谓词或编码结构。

### Lines 286-293
```tablegen
def : Proc<"generic", [FeatureMips32]>;
def : Proc<"mips1", [FeatureMips1]>;
def : Proc<"mips2", [FeatureMips2]>;
def : Proc<"mips32", [FeatureMips32]>;
def : Proc<"mips32r2", [FeatureMips32r2]>;
def : Proc<"mips32r3", [FeatureMips32r3]>;
def : Proc<"mips32r5", [FeatureMips32r5]>;
def : Proc<"mips32r6", [FeatureMips32r6]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 295-308
```tablegen
def : Proc<"mips3", [FeatureMips3]>;
def : Proc<"mips4", [FeatureMips4]>;
def : Proc<"mips5", [FeatureMips5]>;
def : Proc<"mips64", [FeatureMips64]>;
def : Proc<"mips64r2", [FeatureMips64r2]>;
def : Proc<"mips64r3", [FeatureMips64r3]>;
def : Proc<"mips64r5", [FeatureMips64r5]>;
def : Proc<"mips64r6", [FeatureMips64r6]>;
def : Proc<"octeon", [FeatureMips64r2, FeatureCnMips]>;
def : Proc<"octeon+", [FeatureMips64r2, FeatureCnMips, FeatureCnMipsP]>;
def : ProcessorModel<"p5600", MipsP5600Model, [ImplP5600]>;
def : Proc<"r5900", [FeatureR5900]>;
def : ProcessorModel<"i6400", MipsI6400Model, [ImplI6400]>;
def : ProcessorModel<"i6500", MipsI6400Model, [ImplI6500]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 310-312
```tablegen
def MipsAsmParser : AsmParser {
  let ShouldEmitMatchRegisterName = 0;
}
```
- EN: Defines TableGen record `MipsAsmParser` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MipsAsmParser`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 314-315
```tablegen
def MipsAsmParserVariant : AsmParserVariant {
  int Variant = 0;
```
- EN: Defines TableGen record `MipsAsmParserVariant` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MipsAsmParserVariant`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 317-319
```tablegen
  // Recognize hard coded registers.
  string RegisterPrefix = "$";
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 321-323
```tablegen
def MipsAsmWriter : AsmWriter {
  int PassSubtarget = 1;
}
```
- EN: Defines TableGen record `MipsAsmWriter` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MipsAsmWriter`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 325-331
```tablegen
def Mips : Target {
  let InstructionSet = MipsInstrInfo;
  let AssemblyWriters = [MipsAsmWriter];
  let AssemblyParsers = [MipsAsmParser];
  let AssemblyParserVariants = [MipsAsmParserVariant];
  let AllowRegisterRenaming = 1;
}
```
- EN: Defines TableGen record `Mips` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Mips`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 333-335
```tablegen
//===----------------------------------------------------------------------===//
// Pfm Counters
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 337-337
```tablegen
include "MipsPfmCounters.td"
```
- EN: Pulls in other TableGen fragments so this file can reuse common records, predicates, and target-wide definitions.
- CN: 这里引入其他 TableGen 片段，以便复用公共记录、谓词以及整个目标范围内的定义。

## Key Concepts / 关键概念

- EN: Primary role: target descriptions, instruction records, and code-generation metadata.
  - CN: 核心职责：目标描述、指令记录以及代码生成元数据。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `llvm/Target/Target.td`, `MipsRegisterInfo.td`, `MipsInstrInfo.td`, `MipsCallingConv.td`, `MipsRegisterBanks.td`, `MipsCombine.td`, `MipsScheduleI6400.td`, `MipsScheduleP5600.td` ... (+2 more).
  - CN: TableGen 包含项：`llvm/Target/Target.td`, `MipsRegisterInfo.td`, `MipsInstrInfo.td`, `MipsCallingConv.td`, `MipsRegisterBanks.td`, `MipsCombine.td`, `MipsScheduleI6400.td`, `MipsScheduleP5600.td` ... (+2 more)。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
