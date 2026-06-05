# ARMPredicates.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMPredicates.td`
- Repository: `llvm-project`
- Purpose (EN): Defines `ARMPredicates` in LLVM TableGen DSL for the ARM backend, covering target descriptions, instruction records, and code-generation metadata.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 ARM 后端中的 `ARMPredicates`，涵盖目标描述、指令记录以及代码生成元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===-- ARMPredicates.td - ARM Instruction Predicates ------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 9-26
```tablegen
def HasV4T           : Predicate<"Subtarget->hasV4TOps()">,
                                 AssemblerPredicate<(all_of HasV4TOps), "armv4t">;
def NoV4T            : Predicate<"!Subtarget->hasV4TOps()">;
def HasV5T           : Predicate<"Subtarget->hasV5TOps()">,
                                 AssemblerPredicate<(all_of HasV5TOps), "armv5t">;
def NoV5T            : Predicate<"!Subtarget->hasV5TOps()">;
def HasV5TE          : Predicate<"Subtarget->hasV5TEOps()">,
                                 AssemblerPredicate<(all_of HasV5TEOps), "armv5te">;
def HasV6            : Predicate<"Subtarget->hasV6Ops()">,
                                 AssemblerPredicate<(all_of HasV6Ops), "armv6">;
def NoV6             : Predicate<"!Subtarget->hasV6Ops()">;
def HasV6M           : Predicate<"Subtarget->hasV6MOps()">,
                                 AssemblerPredicate<(all_of HasV6MOps),
                                                    "armv6m or armv6t2">;
def HasV8MBaseline   : Predicate<"Subtarget->hasV8MBaselineOps()">,
                                 AssemblerPredicate<(all_of HasV8MBaselineOps),
                                                    "armv8m.base">;
def HasV8MMainline   : Predicate<"Subtarget->hasV8MMainlineOps()">,
```
- EN: Defines TableGen record `HasV4T` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV4T`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 27-44
```tablegen
                                 AssemblerPredicate<(all_of HasV8MMainlineOps),
                                                    "armv8m.main">;
def HasV8_1MMainline : Predicate<"Subtarget->hasV8_1MMainlineOps()">,
                                 AssemblerPredicate<(all_of HasV8_1MMainlineOps),
                                                    "armv8.1m.main">;
def HasMVEInt        : Predicate<"Subtarget->hasMVEIntegerOps()">,
                                 AssemblerPredicate<(all_of HasMVEIntegerOps),
                                                    "mve">;
def HasMVEFloat      : Predicate<"Subtarget->hasMVEFloatOps()">,
                                 AssemblerPredicate<(all_of HasMVEFloatOps),
                                                    "mve.fp">;
def HasCDE           : Predicate<"Subtarget->hasCDEOps()">,
                                 AssemblerPredicate<(all_of HasCDEOps),
                                                    "cde">;
def HasFPRegs        : Predicate<"Subtarget->hasFPRegs()">,
                                 AssemblerPredicate<(all_of FeatureFPRegs),
                                                    "fp registers">;
def HasFPRegs16      : Predicate<"Subtarget->hasFPRegs16()">,
```
- EN: Defines TableGen record `HasV8_1MMainline` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV8_1MMainline`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 45-62
```tablegen
                                 AssemblerPredicate<(all_of FeatureFPRegs16),
                                                    "16-bit fp registers">;
def HasNoFPRegs16    : Predicate<"!Subtarget->hasFPRegs16()">,
                                 AssemblerPredicate<(all_of (not FeatureFPRegs16)),
                                                    "16-bit fp registers">;
def HasFPRegs64      : Predicate<"Subtarget->hasFPRegs64()">,
                                 AssemblerPredicate<(all_of FeatureFPRegs64),
                                                    "64-bit fp registers">;
def HasFPRegsV8_1M   : Predicate<"Subtarget->hasFPRegs() && Subtarget->hasV8_1MMainlineOps()">,
                                 AssemblerPredicate<(all_of FeatureFPRegs, HasV8_1MMainlineOps),
                                                    "armv8.1m.main with FP or MVE">;
def HasV6T2          : Predicate<"Subtarget->hasV6T2Ops()">,
                                 AssemblerPredicate<(all_of HasV6T2Ops), "armv6t2">;
def NoV6T2           : Predicate<"!Subtarget->hasV6T2Ops()">;
def HasV6K           : Predicate<"Subtarget->hasV6KOps()">,
                                 AssemblerPredicate<(all_of HasV6KOps), "armv6k">;
def NoV6K            : Predicate<"!Subtarget->hasV6KOps()">;
def HasV7            : Predicate<"Subtarget->hasV7Ops()">,
```
- EN: Defines TableGen record `HasNoFPRegs16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasNoFPRegs16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 63-80
```tablegen
                                 AssemblerPredicate<(all_of HasV7Ops), "armv7">;
def HasV8            : Predicate<"Subtarget->hasV8Ops()">,
                                 AssemblerPredicate<(all_of HasV8Ops), "armv8">;
def PreV8            : Predicate<"!Subtarget->hasV8Ops()">,
                                 AssemblerPredicate<(all_of (not HasV8Ops)), "armv7 or earlier">;
def HasV8_1a         : Predicate<"Subtarget->hasV8_1aOps()">,
                                 AssemblerPredicate<(all_of HasV8_1aOps), "armv8.1a">;
def HasV8_2a         : Predicate<"Subtarget->hasV8_2aOps()">,
                                 AssemblerPredicate<(all_of HasV8_2aOps), "armv8.2a">;
def HasV8_3a         : Predicate<"Subtarget->hasV8_3aOps()">,
                                 AssemblerPredicate<(all_of HasV8_3aOps), "armv8.3a">;
def HasV8_4a         : Predicate<"Subtarget->hasV8_4aOps()">,
                                 AssemblerPredicate<(all_of HasV8_4aOps), "armv8.4a">;
def HasV8_5a         : Predicate<"Subtarget->hasV8_5aOps()">,
                                 AssemblerPredicate<(all_of HasV8_5aOps), "armv8.5a">;
def HasV8_6a         : Predicate<"Subtarget->hasV8_6aOps()">,
                                 AssemblerPredicate<(all_of HasV8_6aOps), "armv8.6a">;
def HasV8_7a         : Predicate<"Subtarget->hasV8_7aOps()">,
```
- EN: Defines TableGen record `HasV8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 81-98
```tablegen
                                 AssemblerPredicate<(all_of HasV8_7aOps), "armv8.7a">;
def NoVFP            : Predicate<"!Subtarget->hasVFP2Base()">;
def HasVFP2          : Predicate<"Subtarget->hasVFP2Base()">,
                                 AssemblerPredicate<(all_of FeatureVFP2_SP), "VFP2">;
def HasVFP3          : Predicate<"Subtarget->hasVFP3Base()">,
                                 AssemblerPredicate<(all_of FeatureVFP3_D16_SP), "VFP3">;
def HasVFP4          : Predicate<"Subtarget->hasVFP4Base()">,
                                 AssemblerPredicate<(all_of FeatureVFP4_D16_SP), "VFP4">;
def HasDPVFP         : Predicate<"Subtarget->hasFP64()">,
                                 AssemblerPredicate<(all_of FeatureFP64),
                                                    "double precision VFP">;
def HasFPARMv8       : Predicate<"Subtarget->hasFPARMv8Base()">,
                                 AssemblerPredicate<(all_of FeatureFPARMv8_D16_SP), "FPARMv8">;
def HasNEON          : Predicate<"Subtarget->hasNEON()">,
                                 AssemblerPredicate<(all_of FeatureNEON), "NEON">;
def HasSHA2          : Predicate<"Subtarget->hasSHA2()">,
                                 AssemblerPredicate<(all_of FeatureSHA2), "sha2">;
def HasAES           : Predicate<"Subtarget->hasAES()">,
```
- EN: Defines TableGen record `NoVFP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `NoVFP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 99-116
```tablegen
                                 AssemblerPredicate<(all_of FeatureAES), "aes">;
def HasCrypto        : Predicate<"Subtarget->hasCrypto()">,
                                 AssemblerPredicate<(all_of FeatureCrypto), "crypto">;
def HasDotProd       : Predicate<"Subtarget->hasDotProd()">,
                                 AssemblerPredicate<(all_of FeatureDotProd), "dotprod">;
def HasCRC           : Predicate<"Subtarget->hasCRC()">,
                                 AssemblerPredicate<(all_of FeatureCRC), "crc">;
def HasRAS           : Predicate<"Subtarget->hasRAS()">,
                                 AssemblerPredicate<(all_of FeatureRAS), "ras">;
def HasLOB           : Predicate<"Subtarget->hasLOB()">,
                                 AssemblerPredicate<(all_of FeatureLOB), "lob">;
def HasPACBTI        : Predicate<"Subtarget->hasPACBTI()">,
                                 AssemblerPredicate<(all_of FeaturePACBTI), "pacbti">;
def HasFP16          : Predicate<"Subtarget->hasFP16()">,
                                 AssemblerPredicate<(all_of FeatureFP16),"half-float conversions">;
def HasFullFP16      : Predicate<"Subtarget->hasFullFP16()">,
                                 AssemblerPredicate<(all_of FeatureFullFP16),"full half-float">;
def HasFP16FML       : Predicate<"Subtarget->hasFP16FML()">,
```
- EN: Defines TableGen record `HasCrypto` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasCrypto`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 117-134
```tablegen
                                 AssemblerPredicate<(all_of FeatureFP16FML),"full half-float fml">;
def HasBF16          : Predicate<"Subtarget->hasBF16()">,
                                 AssemblerPredicate<(all_of FeatureBF16),"BFloat16 floating point extension">;
def HasMatMulInt8    : Predicate<"Subtarget->hasMatMulInt8()">,
                                 AssemblerPredicate<(all_of FeatureMatMulInt8),"8-bit integer matrix multiply">;
def HasDivideInThumb : Predicate<"Subtarget->hasDivideInThumbMode()">,
                                 AssemblerPredicate<(all_of FeatureHWDivThumb), "divide in THUMB">;
def HasDivideInARM   : Predicate<"Subtarget->hasDivideInARMMode()">,
                                 AssemblerPredicate<(all_of FeatureHWDivARM), "divide in ARM">;
def HasDSP           : Predicate<"Subtarget->hasDSP()">,
                                 AssemblerPredicate<(all_of FeatureDSP), "dsp">;
def HasDB            : Predicate<"Subtarget->hasDataBarrier()">,
                                 AssemblerPredicate<(all_of FeatureDB),
                                                    "data-barriers">;
def HasDFB           : Predicate<"Subtarget->hasFullDataBarrier()">,
                                 AssemblerPredicate<(all_of FeatureDFB),
                                                    "full-data-barrier">;
def HasV7Clrex  : Predicate<"Subtarget->hasV7Clrex()">,
```
- EN: Defines TableGen record `HasBF16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasBF16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 135-152
```tablegen
                            AssemblerPredicate<(all_of FeatureV7Clrex),
                                               "v7 clrex">;
def HasAcquireRelease : Predicate<"Subtarget->hasAcquireRelease()">,
                                  AssemblerPredicate<(all_of FeatureAcquireRelease),
                                                     "acquire/release">;
def HasMP            : Predicate<"Subtarget->hasMPExtension()">,
                                 AssemblerPredicate<(all_of FeatureMP),
                                                    "mp-extensions">;
def HasVirtualization: Predicate<"false">,
                                 AssemblerPredicate<(all_of FeatureVirtualization),
                                                   "virtualization-extensions">;
def HasTrustZone     : Predicate<"Subtarget->hasTrustZone()">,
                                 AssemblerPredicate<(all_of FeatureTrustZone),
                                                    "TrustZone">;
def Has8MSecExt      : Predicate<"Subtarget->has8MSecExt()">,
                                 AssemblerPredicate<(all_of Feature8MSecExt),
                                                    "ARMv8-M Security Extensions">;
def HasZCZ           : Predicate<"Subtarget->hasZeroCycleZeroing()">;
```
- EN: Defines TableGen record `HasAcquireRelease` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasAcquireRelease`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 153-170
```tablegen
def UseNEONForFP     : Predicate<"Subtarget->useNEONForSinglePrecisionFP()">;
def DontUseNEONForFP : Predicate<"!Subtarget->useNEONForSinglePrecisionFP()">;
def IsThumb          : Predicate<"Subtarget->isThumb()">,
                                 AssemblerPredicate<(all_of ModeThumb), "thumb">;
def IsThumb1Only     : Predicate<"Subtarget->isThumb1Only()">;
def IsThumb2         : Predicate<"Subtarget->isThumb2()">,
                                 AssemblerPredicate<(all_of ModeThumb, FeatureThumb2),
                                                    "thumb2">;
def IsMClass         : Predicate<"Subtarget->isMClass()">,
                                 AssemblerPredicate<(all_of FeatureMClass), "armv*m">;
def IsNotMClass      : Predicate<"!Subtarget->isMClass()">,
                                 AssemblerPredicate<(all_of (not FeatureMClass)),
                                                    "!armv*m">;
def IsARM            : Predicate<"!Subtarget->isThumb()">,
                                 AssemblerPredicate<(all_of (not ModeThumb)), "arm-mode">;
def IsMachO          : Predicate<"Subtarget->isTargetMachO()">;
def IsNotMachO       : Predicate<"!Subtarget->isTargetMachO()">;
def IsWindows        : Predicate<"Subtarget->isTargetWindows()">;
```
- EN: Defines TableGen record `UseNEONForFP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `UseNEONForFP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 171-175
```tablegen
def IsNotWindows     : Predicate<"!Subtarget->isTargetWindows()">;
def IsReadTPTPIDRURW : Predicate<"Subtarget->isReadTPTPIDRURW()">;
def IsReadTPTPIDRURO : Predicate<"Subtarget->isReadTPTPIDRURO()">;
def IsReadTPTPIDRPRW : Predicate<"Subtarget->isReadTPTPIDRPRW()">;
def IsReadTPSoft     : Predicate<"Subtarget->isReadTPSoft()">;
```
- EN: Defines TableGen record `IsNotWindows` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IsNotWindows`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 177-180
```tablegen
def UseNegativeImmediates :
  Predicate<"false">,
            AssemblerPredicate<(all_of (not FeatureNoNegativeImmediates)),
                               "NegativeImmediates">;
```
- EN: Defines TableGen record `UseNegativeImmediates` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `UseNegativeImmediates`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 182-187
```tablegen
// FIXME: Eventually this will be just "hasV6T2Ops".
let RecomputePerFunction = 1 in {
  def UseMovt          : Predicate<"Subtarget->useMovt()">;
  def DontUseMovt      : Predicate<"!Subtarget->useMovt()">;
  def UseMovtInPic     : Predicate<"Subtarget->useMovt() && Subtarget->allowPositionIndependentMovt()">;
  def DontUseMovtInPic : Predicate<"!Subtarget->useMovt() || !Subtarget->allowPositionIndependentMovt()">;
```
- EN: Defines TableGen record `UseMovt` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `UseMovt`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 189-193
```tablegen
  def UseFPVMLx: Predicate<"((Subtarget->useFPVMLx() &&"
                           "  TM.Options.AllowFPOpFusion != FPOpFusion::Fast) ||"
                           "Subtarget->hasMinSize())">;
  def SLSBLRMitigation : Predicate<[{ MF->getSubtarget<ARMSubtarget>().hardenSlsBlr() }]>;
  def NoSLSBLRMitigation : Predicate<[{ !MF->getSubtarget<ARMSubtarget>().hardenSlsBlr() }]>;
```
- EN: Defines TableGen record `UseFPVMLx:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `UseFPVMLx:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 195-196
```tablegen
}
def UseMulOps        : Predicate<"Subtarget->useMulOps()">;
```
- EN: Defines TableGen record `UseMulOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `UseMulOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 198-204
```tablegen
// Prefer fused MAC for fp mul + add over fp VMLA / VMLS if they are available.
// But only select them if more precision in FP computation is allowed, and when
// they are not slower than a mul + add sequence.
// Do not use them for Darwin platforms.
def UseFusedMAC      : Predicate<"TM.Options.AllowFPOpFusion =="
                                 "  FPOpFusion::Fast && "
                                 "Subtarget->useFPVFMx()">;
```
- EN: Defines TableGen record `UseFusedMAC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `UseFusedMAC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 206-207
```tablegen
def HasFastVGETLNi32 : Predicate<"!Subtarget->hasSlowVGETLNi32()">;
def HasSlowVGETLNi32 : Predicate<"Subtarget->hasSlowVGETLNi32()">;
```
- EN: Defines TableGen record `HasFastVGETLNi32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasFastVGETLNi32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 209-210
```tablegen
def HasFastVDUP32 : Predicate<"!Subtarget->hasSlowVDUP32()">;
def HasSlowVDUP32 : Predicate<"Subtarget->hasSlowVDUP32()">;
```
- EN: Defines TableGen record `HasFastVDUP32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasFastVDUP32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 212-215
```tablegen
def UseVMOVSR : Predicate<"Subtarget->preferVMOVSR() ||"
                          "!Subtarget->useNEONForSinglePrecisionFP()">;
def DontUseVMOVSR : Predicate<"!Subtarget->preferVMOVSR() &&"
                              "Subtarget->useNEONForSinglePrecisionFP()">;
```
- EN: Defines TableGen record `UseVMOVSR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `UseVMOVSR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 217-220
```tablegen
let RecomputePerFunction = 1 in {
  def IsLE             : Predicate<"MF->getDataLayout().isLittleEndian()">;
  def IsBE             : Predicate<"MF->getDataLayout().isBigEndian()">;
}
```
- EN: Defines TableGen record `IsLE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IsLE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 222-230
```tablegen
def GenExecuteOnly : Predicate<"Subtarget->genExecuteOnly()">;
def DontGenExecuteOnly : Predicate<"!Subtarget->genExecuteOnly()">;
def GenT1ExecuteOnly : Predicate<"Subtarget->genExecuteOnly() && "
                                 "Subtarget->isThumb1Only() && "
                                 "!Subtarget->hasV8MBaselineOps()">;
let RecomputePerFunction = 1 in {
  def SignRetAddr : Predicate<[{ MF->getInfo<ARMFunctionInfo>()->shouldSignReturnAddress(true) }]>;
  def NoSignRetAddr : Predicate<[{ !MF->getInfo<ARMFunctionInfo>()->shouldSignReturnAddress(true) }]>;
}
```
- EN: Defines TableGen record `GenExecuteOnly` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GenExecuteOnly`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 232-234
```tablegen
// Armv8.5-A extensions
def HasSB            : Predicate<"Subtarget->hasSB()">,
                       AssemblerPredicate<(all_of FeatureSB), "sb">;
```
- EN: Defines TableGen record `HasSB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasSB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 236-238
```tablegen
// Armv8.9-A/9.4-A 2022 Architecture extensions
def HasCLRBHB        : Predicate<"Subtarget->hasCLRBHB()">,
                       AssemblerPredicate<(all_of FeatureCLRBHB), "clrbhb">;
```
- EN: Defines TableGen record `HasCLRBHB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasCLRBHB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

## Key Concepts / 关键概念

- EN: Primary role: target descriptions, instruction records, and code-generation metadata.
  - CN: 核心职责：目标描述、指令记录以及代码生成元数据。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `(none)`.
  - CN: TableGen 包含项：`(none)`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
