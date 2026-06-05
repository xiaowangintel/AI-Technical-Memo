# ARMFeatures.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMFeatures.td`
- Repository: `llvm-project`
- Purpose (EN): Defines `ARMFeatures` in LLVM TableGen DSL for the ARM backend, covering feature flags and capability predicates.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 ARM 后端中的 `ARMFeatures`，涵盖特性标志与能力判定。

## Line-by-Line Analysis / 逐行分析

### Lines 1-3
```tablegen
//===----------------------------------------------------------------------===//
// ARM Subtarget state.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 5-7
```tablegen
// True if compiling for Thumb, false for ARM.
def ModeThumb             : SubtargetFeature<"thumb-mode", "IsThumb",
                                             "true", "Thumb mode">;
```
- EN: Defines TableGen record `ModeThumb` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ModeThumb`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 9-12
```tablegen
// True if we're using software floating point features.
def ModeSoftFloat         : SubtargetFeature<"soft-float","UseSoftFloat",
                                             "true", "Use software floating "
                                             "point features.">;
```
- EN: Defines TableGen record `ModeSoftFloat` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ModeSoftFloat`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 14-16
```tablegen
//===----------------------------------------------------------------------===//
// ARM Subtarget features.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 18-28
```tablegen
// This is currently only used by AArch64, but is required here because ARM and
// AArch64 share a tablegen backend for TargetParser.
class Extension<
  string TargetFeatureName,            // String used for -target-feature.
  string Spelling,                     // The XYZ in HasXYZ and AEK_XYZ.
  string Desc,                         // Description.
  list<SubtargetFeature> Implies = []  // List of dependent features.
> : SubtargetFeature<TargetFeatureName, "Has" # Spelling, "true", Desc, Implies>
{
    string ArchExtKindSpelling = "AEK_" # Spelling; // ArchExtKind enum name.
}
```
- EN: Declares reusable TableGen class `Extension` for `ARMFeatures`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMFeatures` 声明可复用的 TableGen 类 `Extension`，通常用于抽象共享字段、谓词或编码结构。

### Lines 30-30
```tablegen
// Floating Point, HW Division and Neon Support
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 32-35
```tablegen
// FP loads/stores/moves, shared between VFP and MVE (even in the integer-only
// version).
def FeatureFPRegs         : SubtargetFeature<"fpregs", "HasFPRegs", "true",
                                             "Enable FP registers">;
```
- EN: Defines TableGen record `FeatureFPRegs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureFPRegs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 37-41
```tablegen
// 16-bit FP loads/stores/moves, shared between VFP (with the v8.2A FP16
// extension) and MVE (even in the integer-only version).
def FeatureFPRegs16       : SubtargetFeature<"fpregs16", "HasFPRegs16", "true",
                                             "Enable 16-bit FP registers",
                                             [FeatureFPRegs]>;
```
- EN: Defines TableGen record `FeatureFPRegs16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureFPRegs16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 43-45
```tablegen
def FeatureFPRegs64       : SubtargetFeature<"fpregs64", "HasFPRegs64", "true",
                                             "Enable 64-bit FP registers",
                                             [FeatureFPRegs]>;
```
- EN: Defines TableGen record `FeatureFPRegs64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureFPRegs64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 47-51
```tablegen
// True if the floating point unit supports double precision.
def FeatureFP64           : SubtargetFeature<"fp64", "HasFP64", "true",
                                             "Floating point unit supports "
                                             "double precision",
                                             [FeatureFPRegs64]>;
```
- EN: Defines TableGen record `FeatureFP64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureFP64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 53-55
```tablegen
// True if subtarget has the full 32 double precision FP registers for VFPv3.
def FeatureD32            : SubtargetFeature<"d32", "HasD32", "true",
                                             "Extend FP to 32 double registers">;
```
- EN: Defines TableGen record `FeatureD32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureD32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 57-74
```tablegen
/// Versions of the VFP flags restricted to single precision, or to
/// 16 d-registers, or both.
multiclass VFPver<string name, string query, string description,
                  list<SubtargetFeature> prev,
                  list<SubtargetFeature> otherimplies,
                  list<SubtargetFeature> vfp2prev = []> {
  def _D16_SP: SubtargetFeature<
    name#"d16sp", query#"D16SP", "true",
    description#" with only 16 d-registers and no double precision",
    !foreach(v, prev, !cast<SubtargetFeature>(v # "_D16_SP")) #
      !foreach(v, vfp2prev, !cast<SubtargetFeature>(v # "_SP")) #
      otherimplies>;
  def _SP: SubtargetFeature<
    name#"sp", query#"SP", "true",
    description#" with no double precision",
    !foreach(v, prev, !cast<SubtargetFeature>(v # "_SP")) #
      otherimplies # [FeatureD32, !cast<SubtargetFeature>(NAME # "_D16_SP")]>;
  def _D16: SubtargetFeature<
```
- EN: Declares TableGen `multiclass VFPver`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass VFPver`，它是一个可复用模板，可展开为多个相关记录。

### Lines 75-85
```tablegen
    name#"d16", query#"D16", "true",
    description#" with only 16 d-registers",
    !foreach(v, prev, !cast<SubtargetFeature>(v # "_D16")) #
      vfp2prev #
      otherimplies # [FeatureFP64, !cast<SubtargetFeature>(NAME # "_D16_SP")]>;
  def "": SubtargetFeature<
    name, query, "true", description,
    prev # otherimplies # [
        !cast<SubtargetFeature>(NAME # "_D16"),
        !cast<SubtargetFeature>(NAME # "_SP")]>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 87-90
```tablegen
def FeatureVFP2_SP        : SubtargetFeature<"vfp2sp", "HasVFPv2SP", "true",
                                             "Enable VFP2 instructions with "
                                             "no double precision",
                                             [FeatureFPRegs]>;
```
- EN: Defines TableGen record `FeatureVFP2_SP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureVFP2_SP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 92-94
```tablegen
def FeatureVFP2           : SubtargetFeature<"vfp2", "HasVFPv2", "true",
                                             "Enable VFP2 instructions",
                                             [FeatureFP64, FeatureVFP2_SP]>;
```
- EN: Defines TableGen record `FeatureVFP2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureVFP2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 96-97
```tablegen
defm FeatureVFP3: VFPver<"vfp3", "HasVFPv3", "Enable VFP3 instructions",
                         [], [], [FeatureVFP2]>;
```
- EN: Defines TableGen record `FeatureVFP3:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureVFP3:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 99-101
```tablegen
def FeatureNEON           : SubtargetFeature<"neon", "HasNEON", "true",
                                             "Enable NEON instructions",
                                             [FeatureVFP3]>;
```
- EN: Defines TableGen record `FeatureNEON` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureNEON`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 103-106
```tablegen
// True if subtarget supports half-precision FP conversions.
def FeatureFP16           : SubtargetFeature<"fp16", "HasFP16", "true",
                                             "Enable half-precision "
                                             "floating point">;
```
- EN: Defines TableGen record `FeatureFP16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureFP16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 108-109
```tablegen
defm FeatureVFP4: VFPver<"vfp4", "HasVFPv4", "Enable VFP4 instructions",
                         [FeatureVFP3], [FeatureFP16]>;
```
- EN: Defines TableGen record `FeatureVFP4:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureVFP4:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 111-112
```tablegen
defm FeatureFPARMv8: VFPver<"fp-armv8", "HasFPARMv8", "Enable ARMv8 FP",
                         [FeatureVFP4], []>;
```
- EN: Defines TableGen record `FeatureFPARMv8:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureFPARMv8:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 114-118
```tablegen
// True if subtarget supports half-precision FP operations.
def FeatureFullFP16       : SubtargetFeature<"fullfp16", "HasFullFP16", "true",
                                             "Enable full half-precision "
                                             "floating point",
                                             [FeatureFPARMv8_D16_SP, FeatureFPRegs16]>;
```
- EN: Defines TableGen record `FeatureFullFP16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureFullFP16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 120-124
```tablegen
// True if subtarget supports half-precision FP fml operations.
def FeatureFP16FML        : SubtargetFeature<"fp16fml", "HasFP16FML", "true",
                                             "Enable full half-precision "
                                             "floating point fml instructions",
                                             [FeatureFullFP16]>;
```
- EN: Defines TableGen record `FeatureFP16FML` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureFP16FML`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 126-129
```tablegen
// True if subtarget supports [su]div in Thumb mode.
def FeatureHWDivThumb     : SubtargetFeature<"hwdiv",
                                             "HasDivideInThumbMode", "true",
                                             "Enable divide instructions in Thumb">;
```
- EN: Defines TableGen record `FeatureHWDivThumb` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureHWDivThumb`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 131-134
```tablegen
// True if subtarget supports [su]div in ARM mode.
def FeatureHWDivARM       : SubtargetFeature<"hwdiv-arm",
                                             "HasDivideInARMMode", "true",
                                             "Enable divide instructions in ARM mode">;
```
- EN: Defines TableGen record `FeatureHWDivARM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureHWDivARM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 136-136
```tablegen
// Atomic Support
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 138-140
```tablegen
// True if the subtarget supports DMB / DSB data barrier instructions.
def FeatureDB             : SubtargetFeature<"db", "HasDataBarrier", "true",
                                             "Has data barrier (dmb/dsb) instructions">;
```
- EN: Defines TableGen record `FeatureDB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureDB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 142-144
```tablegen
// True if the subtarget supports CLREX instructions.
def FeatureV7Clrex        : SubtargetFeature<"v7clrex", "HasV7Clrex", "true",
                                             "Has v7 clrex instruction">;
```
- EN: Defines TableGen record `FeatureV7Clrex` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureV7Clrex`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 146-148
```tablegen
// True if the subtarget supports DFB data barrier instruction.
def FeatureDFB  : SubtargetFeature<"dfb", "HasFullDataBarrier", "true",
                                   "Has full data barrier (dfb) instruction">;
```
- EN: Defines TableGen record `FeatureDFB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureDFB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 150-154
```tablegen
// True if the subtarget supports v8 atomics (LDA/LDAEX etc) instructions.
def FeatureAcquireRelease : SubtargetFeature<"acquire-release",
                                             "HasAcquireRelease", "true",
                                             "Has v8 acquire/release (lda/ldaex "
                                             " etc) instructions">;
```
- EN: Defines TableGen record `FeatureAcquireRelease` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureAcquireRelease`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 157-159
```tablegen
// True if floating point compare + branch is slow.
def FeatureSlowFPBrcc     : SubtargetFeature<"slow-fp-brcc", "IsFPBrccSlow", "true",
                                             "FP compare + branch is slow">;
```
- EN: Defines TableGen record `FeatureSlowFPBrcc` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureSlowFPBrcc`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 161-166
```tablegen
// True if the processor supports the Performance Monitor Extensions. These
// include a generic cycle-counter as well as more fine-grained (often
// implementation-specific) events.
def FeaturePerfMon        : SubtargetFeature<"perfmon", "HasPerfMon", "true",
                                             "Enable support for Performance "
                                             "Monitor extensions">;
```
- EN: Defines TableGen record `FeaturePerfMon` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeaturePerfMon`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 169-169
```tablegen
// TrustZone Security Extensions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 171-174
```tablegen
// True if processor supports TrustZone security extensions.
def FeatureTrustZone      : SubtargetFeature<"trustzone", "HasTrustZone", "true",
                                             "Enable support for TrustZone "
                                             "security extensions">;
```
- EN: Defines TableGen record `FeatureTrustZone` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureTrustZone`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 176-179
```tablegen
// True if processor supports ARMv8-M Security Extensions.
def Feature8MSecExt       : SubtargetFeature<"8msecext", "Has8MSecExt", "true",
                                             "Enable support for ARMv8-M "
                                             "Security Extensions">;
```
- EN: Defines TableGen record `Feature8MSecExt` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Feature8MSecExt`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 181-183
```tablegen
// True if processor supports SHA1 and SHA256.
def FeatureSHA2           : SubtargetFeature<"sha2", "HasSHA2", "true",
                                             "Enable SHA1 and SHA256 support", [FeatureNEON]>;
```
- EN: Defines TableGen record `FeatureSHA2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureSHA2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 185-186
```tablegen
def FeatureAES            : SubtargetFeature<"aes", "HasAES", "true",
                                             "Enable AES support", [FeatureNEON]>;
```
- EN: Defines TableGen record `FeatureAES` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureAES`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 188-192
```tablegen
// True if processor supports Cryptography extensions.
def FeatureCrypto         : SubtargetFeature<"crypto", "HasCrypto", "true",
                                             "Enable support for "
                                             "Cryptography extensions",
                                             [FeatureNEON, FeatureSHA2, FeatureAES]>;
```
- EN: Defines TableGen record `FeatureCrypto` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureCrypto`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 194-196
```tablegen
// True if processor supports CRC instructions.
def FeatureCRC            : SubtargetFeature<"crc", "HasCRC", "true",
                                             "Enable support for CRC instructions">;
```
- EN: Defines TableGen record `FeatureCRC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureCRC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 198-201
```tablegen
// True if the ARMv8.2A dot product instructions are supported.
def FeatureDotProd        : SubtargetFeature<"dotprod", "HasDotProd", "true",
                                             "Enable support for dot product instructions",
                                             [FeatureNEON]>;
```
- EN: Defines TableGen record `FeatureDotProd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureDotProd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 203-207
```tablegen
// True if the processor supports RAS extensions.
// Not to be confused with FeatureHasRetAddrStack (return address stack).
def FeatureRAS            : SubtargetFeature<"ras", "HasRAS", "true",
                                             "Enable Reliability, Availability "
                                             "and Serviceability extensions">;
```
- EN: Defines TableGen record `FeatureRAS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureRAS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 209-213
```tablegen
// Fast computation of non-negative address offsets.
// True if processor does positive address offset computation faster.
def FeatureFPAO           : SubtargetFeature<"fpao", "HasFPAO", "true",
                                             "Enable fast computation of "
                                             "positive address offsets">;
```
- EN: Defines TableGen record `FeatureFPAO` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureFPAO`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 215-218
```tablegen
// Fast execution of AES crypto operations.
// True if processor executes back to back AES instruction pairs faster.
def FeatureFuseAES        : SubtargetFeature<"fuse-aes", "HasFuseAES", "true",
                                             "CPU fuses AES crypto operations">;
```
- EN: Defines TableGen record `FeatureFuseAES` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureFuseAES`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 220-223
```tablegen
// Fast execution of bottom and top halves of literal generation.
// True if processor executes back to back bottom and top halves of literal generation faster.
def FeatureFuseLiterals   : SubtargetFeature<"fuse-literals", "HasFuseLiterals", "true",
                                             "CPU fuses literal generation operations">;
```
- EN: Defines TableGen record `FeatureFuseLiterals` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureFuseLiterals`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 225-231
```tablegen
// Choice of hardware register to use as the thread pointer, if any.
def FeatureReadTpTPIDRURW :  SubtargetFeature<"read-tp-tpidrurw", "IsReadTPTPIDRURW", "true",
                                      "Reading thread pointer from TPIDRURW register">;
def FeatureReadTpTPIDRURO :  SubtargetFeature<"read-tp-tpidruro", "IsReadTPTPIDRURO", "true",
                                      "Reading thread pointer from TPIDRURO register">;
def FeatureReadTpTPIDRPRW :  SubtargetFeature<"read-tp-tpidrprw", "IsReadTPTPIDRPRW", "true",
                                      "Reading thread pointer from TPIDRPRW register">;
```
- EN: Defines TableGen record `FeatureReadTpTPIDRURW` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureReadTpTPIDRURW`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 233-237
```tablegen
// Cyclone can zero VFP registers in 0 cycles.
// True if the instructions "vmov.i32 d0, #0" and "vmov.i32 q0, #0" are
// particularly effective at zeroing a VFP register.
def FeatureZCZeroing      : SubtargetFeature<"zcz", "HasZeroCycleZeroing", "true",
                                             "Has zero-cycle zeroing instructions">;
```
- EN: Defines TableGen record `FeatureZCZeroing` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureZCZeroing`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 239-243
```tablegen
// Whether it is profitable to unpredicate certain instructions during if-conversion.
// True if if conversion may decide to leave some instructions unpredicated.
def FeatureProfUnpredicate : SubtargetFeature<"prof-unpr",
                                              "IsProfitableToUnpredicate", "true",
                                              "Is profitable to unpredicate">;
```
- EN: Defines TableGen record `FeatureProfUnpredicate` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureProfUnpredicate`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 245-249
```tablegen
// Some targets (e.g. Swift) have microcoded VGETLNi32.
// True if VMOV will be favored over VGETLNi32.
def FeatureSlowVGETLNi32  : SubtargetFeature<"slow-vgetlni32",
                                             "HasSlowVGETLNi32", "true",
                                             "Has slow VGETLNi32 - prefer VMOV">;
```
- EN: Defines TableGen record `FeatureSlowVGETLNi32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureSlowVGETLNi32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 251-255
```tablegen
// Some targets (e.g. Swift) have microcoded VDUP32.
// True if VMOV will be favored over VDUP.
def FeatureSlowVDUP32     : SubtargetFeature<"slow-vdup32", "HasSlowVDUP32",
                                             "true",
                                             "Has slow VDUP32 - prefer VMOV">;
```
- EN: Defines TableGen record `FeatureSlowVDUP32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureSlowVDUP32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 257-261
```tablegen
// Some targets (e.g. Cortex-A9) prefer VMOVSR to VMOVDRR even when using NEON
// for scalar FP, as this allows more effective execution domain optimization.
// True if VMOVSR will be favored over VMOVDRR.
def FeaturePreferVMOVSR   : SubtargetFeature<"prefer-vmovsr", "PreferVMOVSR",
                                             "true", "Prefer VMOVSR">;
```
- EN: Defines TableGen record `FeaturePreferVMOVSR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeaturePreferVMOVSR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 263-267
```tablegen
// Swift has ISHST barriers compatible with Atomic Release semantics but weaker
// than ISH.
// True if ISHST barriers will be used for Release semantics.
def FeaturePrefISHSTBarrier : SubtargetFeature<"prefer-ishst", "PreferISHSTBarriers",
                                               "true", "Prefer ISHST barriers">;
```
- EN: Defines TableGen record `FeaturePrefISHSTBarrier` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeaturePrefISHSTBarrier`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 269-273
```tablegen
// Some targets (e.g. Cortex-A9) have muxed AGU and NEON/FPU.
// True if the AGU and NEON/FPU units are multiplexed.
def FeatureMuxedUnits     : SubtargetFeature<"muxed-units", "HasMuxedUnits",
                                             "true",
                                             "Has muxed AGU and NEON/FPU">;
```
- EN: Defines TableGen record `FeatureMuxedUnits` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureMuxedUnits`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 275-281
```tablegen
// Whether VLDM/VSTM starting with odd register number need more microops
// than single VLDRS.
// True if a VLDM/VSTM starting with an odd register number is considered to
// take more microops than single VLDRS/VSTRS.
def FeatureSlowOddRegister : SubtargetFeature<"slow-odd-reg", "HasSlowOddRegister",
                                              "true", "VLDM/VSTM starting "
                                              "with an odd register is slow">;
```
- EN: Defines TableGen record `FeatureSlowOddRegister` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureSlowOddRegister`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 283-287
```tablegen
// Some targets have a renaming dependency when loading into D subregisters.
// True if loading into a D subregister will be penalized.
def FeatureSlowLoadDSubreg : SubtargetFeature<"slow-load-D-subreg",
                                              "HasSlowLoadDSubregister", "true",
                                              "Loading into D subregs is slow">;
```
- EN: Defines TableGen record `FeatureSlowLoadDSubreg` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureSlowLoadDSubreg`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 289-292
```tablegen
// True if use a wider stride when allocating VFP registers.
def FeatureUseWideStrideVFP : SubtargetFeature<"wide-stride-vfp",
                                               "UseWideStrideVFP", "true",
                                               "Use a wide stride when allocating VFP registers">;
```
- EN: Defines TableGen record `FeatureUseWideStrideVFP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureUseWideStrideVFP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 294-298
```tablegen
// Some targets (e.g. Cortex-A15) never want VMOVS to be widened to VMOVD.
// True if VMOVS will never be widened to VMOVD.
def FeatureDontWidenVMOVS : SubtargetFeature<"dont-widen-vmovs",
                                             "DontWidenVMOVS", "true",
                                             "Don't widen VMOVS to VMOVD">;
```
- EN: Defines TableGen record `FeatureDontWidenVMOVS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureDontWidenVMOVS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 300-306
```tablegen
// Some targets (e.g. Cortex-A15) prefer to avoid mixing operations on different
// VFP register widths.
// True if splat a register between VFP and NEON instructions.
def FeatureSplatVFPToNeon : SubtargetFeature<"splat-vfp-neon",
                                             "UseSplatVFPToNeon", "true",
                                             "Splat register from VFP to NEON",
                                             [FeatureDontWidenVMOVS]>;
```
- EN: Defines TableGen record `FeatureSplatVFPToNeon` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureSplatVFPToNeon`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 308-312
```tablegen
// Whether or not it is profitable to expand VFP/NEON MLA/MLS instructions.
// True if run the MLx expansion pass.
def FeatureExpandMLx      : SubtargetFeature<"expand-fp-mlx",
                                             "ExpandMLx", "true",
                                             "Expand VFP/NEON MLA/MLS instructions">;
```
- EN: Defines TableGen record `FeatureExpandMLx` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureExpandMLx`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 314-317
```tablegen
// Some targets have special RAW hazards for VFP/NEON VMLA/VMLS.
// True if VFP/NEON VMLA/VMLS have special RAW hazards.
def FeatureHasVMLxHazards : SubtargetFeature<"vmlx-hazards", "HasVMLxHazards",
                                             "true", "Has VMLx hazards">;
```
- EN: Defines TableGen record `FeatureHasVMLxHazards` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureHasVMLxHazards`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 319-325
```tablegen
// Some targets (e.g. Cortex-A9) want to convert VMOVRS, VMOVSR and VMOVS from
// VFP to NEON, as an execution domain optimization.
// True if VMOVRS, VMOVSR and VMOVS will be converted from VFP to NEON.
def FeatureNEONForFPMovs  : SubtargetFeature<"neon-fpmovs",
                                             "UseNEONForFPMovs", "true",
                                             "Convert VMOVSR, VMOVRS, "
                                             "VMOVS to NEON">;
```
- EN: Defines TableGen record `FeatureNEONForFPMovs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureNEONForFPMovs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 327-334
```tablegen
// Some processors benefit from using NEON instructions for scalar
// single-precision FP operations. This affects instruction selection and should
// only be enabled if the handling of denormals is not important.
// Use the method useNEONForSinglePrecisionFP() to determine if NEON should actually be used.
def FeatureNEONForFP      : SubtargetFeature<"neonfp",
                                             "HasNEONForFP",
                                             "true",
                                             "Use NEON for single precision FP">;
```
- EN: Defines TableGen record `FeatureNEONForFP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureNEONForFP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 336-341
```tablegen
// On some processors, VLDn instructions that access unaligned data take one
// extra cycle. Take that into account when computing operand latencies.
// True if VLDn instructions take an extra cycle for unaligned accesses.
def FeatureCheckVLDnAlign : SubtargetFeature<"vldn-align", "CheckVLDnAccessAlignment",
                                             "true",
                                             "Check for VLDn unaligned access">;
```
- EN: Defines TableGen record `FeatureCheckVLDnAlign` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureCheckVLDnAlign`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 343-347
```tablegen
// Some processors have a nonpipelined VFP coprocessor.
// True if VFP instructions are not pipelined.
def FeatureNonpipelinedVFP : SubtargetFeature<"nonpipelined-vfp",
                                              "NonpipelinedVFP", "true",
                                              "VFP instructions are not pipelined">;
```
- EN: Defines TableGen record `FeatureNonpipelinedVFP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureNonpipelinedVFP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 349-355
```tablegen
// Some processors have FP multiply-accumulate instructions that don't
// play nicely with other VFP / NEON instructions, and it's generally better
// to just not use them.
// If the VFP2 / NEON instructions are available, indicates
// whether the FP VML[AS] instructions are slow (if so, don't use them).
def FeatureHasSlowFPVMLx  : SubtargetFeature<"slowfpvmlx", "SlowFPVMLx", "true",
                                             "Disable VFP / NEON MAC instructions">;
```
- EN: Defines TableGen record `FeatureHasSlowFPVMLx` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureHasSlowFPVMLx`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 357-361
```tablegen
// VFPv4 added VFMA instructions that can similarly be fast or slow.
// If the VFP4 / NEON instructions are available, indicates
// whether the FP VFM[AS] instructions are slow (if so, don't use them).
def FeatureHasSlowFPVFMx  : SubtargetFeature<"slowfpvfmx", "SlowFPVFMx", "true",
                                             "Disable VFP / NEON FMA instructions">;
```
- EN: Defines TableGen record `FeatureHasSlowFPVFMx` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureHasSlowFPVFMx`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 363-368
```tablegen
// Cortex-A8 / A9 Advanced SIMD has multiplier accumulator forwarding.
/// True if NEON has special multiplier accumulator
/// forwarding to allow mul + mla being issued back to back.
def FeatureVMLxForwarding : SubtargetFeature<"vmlx-forwarding",
                                             "HasVMLxForwarding", "true",
                                             "Has multiplier accumulator forwarding">;
```
- EN: Defines TableGen record `FeatureVMLxForwarding` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureVMLxForwarding`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 370-373
```tablegen
// Disable 32-bit to 16-bit narrowing for experimentation.
// True if codegen would prefer 32-bit Thumb instructions over 16-bit ones.
def FeaturePref32BitThumb : SubtargetFeature<"32bit", "Prefers32BitThumb", "true",
                                             "Prefer 32-bit Thumb instrs">;
```
- EN: Defines TableGen record `FeaturePref32BitThumb` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeaturePref32BitThumb`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 375-376
```tablegen
def FeaturePreferBranchAlign32 : SubtargetFeature<"loop-align", "PreferBranchLogAlignment","2",
                                              "Prefer 32-bit alignment for branch targets">;
```
- EN: Defines TableGen record `FeaturePreferBranchAlign32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeaturePreferBranchAlign32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 378-379
```tablegen
def FeaturePreferBranchAlign64 : SubtargetFeature<"branch-align-64", "PreferBranchLogAlignment","3",
                                              "Prefer 64-bit alignment for branch targets">;
```
- EN: Defines TableGen record `FeaturePreferBranchAlign64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeaturePreferBranchAlign64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 381-382
```tablegen
def FeatureMVEVectorCostFactor1 : SubtargetFeature<"mve1beat", "MVEVectorCostFactor", "4",
                        "Model MVE instructions as a 1 beat per tick architecture">;
```
- EN: Defines TableGen record `FeatureMVEVectorCostFactor1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureMVEVectorCostFactor1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 384-385
```tablegen
def FeatureMVEVectorCostFactor2 : SubtargetFeature<"mve2beat", "MVEVectorCostFactor", "2",
                        "Model MVE instructions as a 2 beats per tick architecture">;
```
- EN: Defines TableGen record `FeatureMVEVectorCostFactor2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureMVEVectorCostFactor2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 387-388
```tablegen
def FeatureMVEVectorCostFactor4 : SubtargetFeature<"mve4beat", "MVEVectorCostFactor", "1",
                        "Model MVE instructions as a 4 beats per tick architecture">;
```
- EN: Defines TableGen record `FeatureMVEVectorCostFactor4` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureMVEVectorCostFactor4`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 390-399
```tablegen
/// Some instructions update CPSR partially, which can add false dependency for
/// out-of-order implementation, e.g. Cortex-A9, unless each individual bit is
/// mapped to a separate physical register. Avoid partial CPSR update for these
/// processors.
/// True if codegen would avoid using instructions
/// that partially update CPSR and add false dependency on the previous
/// CPSR setting instruction.
def FeatureAvoidPartialCPSR : SubtargetFeature<"avoid-partial-cpsr",
                                               "AvoidCPSRPartialUpdate", "true",
                                 "Avoid CPSR partial update for OOO execution">;
```
- EN: Defines TableGen record `FeatureAvoidPartialCPSR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureAvoidPartialCPSR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 401-405
```tablegen
/// FeatureAvoidMULS - If true, codegen would avoid using the MULS instruction,
/// prefering the thumb2 MUL which doesn't set flags.
def FeatureAvoidMULS : SubtargetFeature<"avoid-muls",
                                        "AvoidMULS", "true",
                                 "Avoid MULS instructions for M class cores">;
```
- EN: Declares reusable TableGen class `cores` for `ARMFeatures`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMFeatures` 声明可复用的 TableGen 类 `cores`，通常用于抽象共享字段、谓词或编码结构。

### Lines 408-414
```tablegen
/// Disable +1 predication cost for instructions updating CPSR.
/// Enabled for Cortex-A57.
/// True if disable +1 predication cost for instructions updating CPSR. Enabled for Cortex-A57.
def FeatureCheapPredicableCPSR : SubtargetFeature<"cheap-predicable-cpsr",
                                                  "CheapPredicableCPSRDef",
                                                  "true",
                  "Disable +1 predication cost for instructions updating CPSR">;
```
- EN: Defines TableGen record `FeatureCheapPredicableCPSR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureCheapPredicableCPSR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 416-420
```tablegen
// True if codegen should avoid using flag setting movs with shifter operand (i.e. asr, lsl, lsr).
def FeatureAvoidMOVsShOp  : SubtargetFeature<"avoid-movs-shop",
                                             "AvoidMOVsShifterOperand", "true",
                                             "Avoid movs instructions with "
                                             "shifter operand">;
```
- EN: Defines TableGen record `FeatureAvoidMOVsShOp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureAvoidMOVsShOp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 422-426
```tablegen
// Some processors perform return stack prediction. CodeGen should avoid issue
// "normal" call instructions to callees which do not return.
def FeatureHasRetAddrStack : SubtargetFeature<"ret-addr-stack",
                                              "HasRetAddrStack", "true",
                                              "Has return address stack">;
```
- EN: Defines TableGen record `FeatureHasRetAddrStack` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureHasRetAddrStack`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 428-436
```tablegen
// Some processors have no branch predictor, which changes the expected cost of
// taking a branch which affects the choice of whether to use predicated
// instructions.
// True if the subtarget has a branch predictor. Having
// a branch predictor or not changes the expected cost of taking a branch
// which affects the choice of whether to use predicated instructions.
def FeatureHasNoBranchPredictor : SubtargetFeature<"no-branch-predictor",
                                                   "HasBranchPredictor", "false",
                                                   "Has no branch predictor">;
```
- EN: Defines TableGen record `FeatureHasNoBranchPredictor` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureHasNoBranchPredictor`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 438-442
```tablegen
/// DSP extension.
/// True if the subtarget supports the DSP (saturating arith and such) instructions.
def FeatureDSP            : SubtargetFeature<"dsp", "HasDSP", "true",
                                             "Supports DSP instructions in "
                                             "ARM and/or Thumb2">;
```
- EN: Defines TableGen record `FeatureDSP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureDSP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 444-446
```tablegen
// True if the subtarget supports Multiprocessing extension (ARMv7 only).
def FeatureMP             : SubtargetFeature<"mp", "HasMPExtension", "true",
                                        "Supports Multiprocessing extension">;
```
- EN: Defines TableGen record `FeatureMP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureMP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 448-452
```tablegen
// Virtualization extension - requires HW divide (ARMv7-AR ARMARM - 4.4.8).
def FeatureVirtualization : SubtargetFeature<"virtualization",
                                             "HasVirtualization", "true",
                                             "Supports Virtualization extension",
                                             [FeatureHWDivThumb, FeatureHWDivARM]>;
```
- EN: Defines TableGen record `FeatureVirtualization` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureVirtualization`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 454-460
```tablegen
// True if the subtarget disallows unaligned memory
// accesses for some types.  For details, see
// ARMTargetLowering::allowsMisalignedMemoryAccesses().
def FeatureStrictAlign    : SubtargetFeature<"strict-align",
                                             "StrictAlign", "true",
                                             "Disallow all unaligned memory "
                                             "access">;
```
- EN: Defines TableGen record `FeatureStrictAlign` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureStrictAlign`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 462-465
```tablegen
// Generate calls via indirect call instructions.
def FeatureLongCalls      : SubtargetFeature<"long-calls", "GenLongCalls", "true",
                                             "Generate calls via indirect call "
                                             "instructions">;
```
- EN: Defines TableGen record `FeatureLongCalls` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureLongCalls`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 467-471
```tablegen
// Generate code that does not contain data access to code sections.
def FeatureExecuteOnly    : SubtargetFeature<"execute-only",
                                             "GenExecuteOnly", "true",
                                             "Enable the generation of "
                                             "execute only code.">;
```
- EN: Defines TableGen record `FeatureExecuteOnly` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureExecuteOnly`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 473-476
```tablegen
// True if R9 is not available as a general purpose register.
def FeatureReserveR9      : SubtargetFeature<"reserve-r9", "ReserveR9", "true",
                                             "Reserve R9, making it unavailable"
                                             " as GPR">;
```
- EN: Defines TableGen record `FeatureReserveR9` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureReserveR9`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 478-482
```tablegen
// True if MOVT / MOVW pairs are not used for materialization of
// 32-bit imms (including global addresses).
def FeatureNoMovt         : SubtargetFeature<"no-movt", "NoMovt", "true",
                                             "Don't use movt/movw pairs for "
                                             "32-bit imms">;
```
- EN: Defines TableGen record `FeatureNoMovt` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureNoMovt`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 484-492
```tablegen
/// Implicitly convert an instruction to a different one if its immediates
/// cannot be encoded. For example, ADD r0, r1, #FFFFFFFF -> SUB r0, r1, #1.
def FeatureNoNegativeImmediates
                          : SubtargetFeature<"no-neg-immediates",
                                             "NegativeImmediates", "false",
                                             "Convert immediates and instructions "
                                             "to their negated or complemented "
                                             "equivalent when the immediate does "
                                             "not fit in the encoding.">;
```
- EN: Defines TableGen record `FeatureNoNegativeImmediates` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureNoNegativeImmediates`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 494-496
```tablegen
// Use the MachineScheduler for instruction scheduling for the subtarget.
def FeatureUseMISched: SubtargetFeature<"use-misched", "UseMISched", "true",
                                        "Use the MachineScheduler">;
```
- EN: Defines TableGen record `FeatureUseMISched:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureUseMISched:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 498-500
```tablegen
// Use the MachinePipeliner for instruction scheduling for the subtarget.
def FeatureUseMIPipeliner: SubtargetFeature<"use-mipipeliner", "UseMIPipeliner", "true",
                                            "Use the MachinePipeliner">;
```
- EN: Defines TableGen record `FeatureUseMIPipeliner:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureUseMIPipeliner:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 502-505
```tablegen
// False if scheduling should happen again after register allocation.
def FeatureNoPostRASched : SubtargetFeature<"disable-postra-scheduler",
    "DisablePostRAScheduler", "true",
    "Don't schedule again after register allocation">;
```
- EN: Defines TableGen record `FeatureNoPostRASched` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureNoPostRASched`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 507-507
```tablegen
// Armv8.5-A extensions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 509-511
```tablegen
// Has speculation barrier.
def FeatureSB       : SubtargetFeature<"sb", "HasSB", "true",
  "Enable v8.5a Speculation Barrier" >;
```
- EN: Defines TableGen record `FeatureSB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureSB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 513-513
```tablegen
// Armv8.6-A extensions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 515-517
```tablegen
// True if subtarget supports BFloat16 floating point operations.
def FeatureBF16     : SubtargetFeature<"bf16", "HasBF16", "true",
  "Enable support for BFloat16 instructions",  [FeatureNEON]>;
```
- EN: Defines TableGen record `FeatureBF16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureBF16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 519-521
```tablegen
// True if subtarget supports 8-bit integer matrix multiply.
def FeatureMatMulInt8 : SubtargetFeature<"i8mm", "HasMatMulInt8",
    "true", "Enable Matrix Multiply Int8 Extension", [FeatureNEON]>;
```
- EN: Defines TableGen record `FeatureMatMulInt8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureMatMulInt8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 523-523
```tablegen
// Armv8.1-M extensions
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 525-528
```tablegen
// True if the processor supports the Low Overhead Branch extension.
def FeatureLOB            : SubtargetFeature<"lob", "HasLOB", "true",
                                             "Enable Low Overhead Branch "
                                             "extensions">;
```
- EN: Defines TableGen record `FeatureLOB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureLOB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 530-534
```tablegen
// Mitigate against the cve-2021-35465 security vulnurability.
def FeatureFixCMSE_CVE_2021_35465 : SubtargetFeature<"fix-cmse-cve-2021-35465",
                                        "FixCMSE_CVE_2021_35465", "true",
                                        "Mitigate against the cve-2021-35465 "
                                        "security vulnurability">;
```
- EN: Defines TableGen record `FeatureFixCMSE_CVE_2021_35465` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureFixCMSE_CVE_2021_35465`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 536-538
```tablegen
def FeaturePACBTI         : SubtargetFeature<"pacbti", "HasPACBTI", "true",
                                             "Enable Pointer Authentication and Branch "
                                             "Target Identification">;
```
- EN: Defines TableGen record `FeaturePACBTI` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeaturePACBTI`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 540-544
```tablegen
/// Don't place a BTI instruction after return-twice constructs (setjmp).
def FeatureNoBTIAtReturnTwice : SubtargetFeature<"no-bti-at-return-twice",
                                                 "NoBTIAtReturnTwice", "true",
                                                 "Don't place a BTI instruction "
                                                 "after a return-twice">;
```
- EN: Defines TableGen record `FeatureNoBTIAtReturnTwice` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureNoBTIAtReturnTwice`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 546-548
```tablegen
// Armv8.9-A/Armv9.4-A 2022 Architecture Extensions
def FeatureCLRBHB : SubtargetFeature<"clrbhb", "HasCLRBHB", "true",
                                     "Enable Clear BHB instruction">;
```
- EN: Defines TableGen record `FeatureCLRBHB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureCLRBHB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 551-553
```tablegen
def FeatureFixCortexA57AES1742098 : SubtargetFeature<"fix-cortex-a57-aes-1742098",
  "FixCortexA57AES1742098", "true",
  "Work around Cortex-A57 Erratum 1742098 / Cortex-A72 Erratum 1655431 (AES)">;
```
- EN: Defines TableGen record `FeatureFixCortexA57AES1742098` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureFixCortexA57AES1742098`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 555-562
```tablegen
// If frame pointers are in use, they must follow the AAPCS definition, which
// always uses R11 as the frame pointer. If this is not set, we can use R7 as
// the frame pointer for Thumb1-only code, which is more efficient, but less
// compatible. Note that this feature does not control whether frame pointers
// are emitted, that is controlled by the "frame-pointer" function attribute.
def FeatureAAPCSFrameChain : SubtargetFeature<"aapcs-frame-chain",
                                              "CreateAAPCSFrameChain", "true",
                                              "Create an AAPCS compliant frame chain">;
```
- EN: Defines TableGen record `FeatureAAPCSFrameChain` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureAAPCSFrameChain`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 564-571
```tablegen
// Assume that lock-free 32-bit atomics are available, even if the target
// and operating system combination would not usually provide them. The user
// is responsible for providing any necessary __sync implementations. Code
// built with this feature is not ABI-compatible with code built without this
// feature, if atomic variables are exposed across the ABI boundary.
def FeatureAtomics32 : SubtargetFeature<
    "atomics-32", "HasForced32BitAtomics", "true",
    "Assume that lock-free 32-bit atomics are available">;
```
- EN: Defines TableGen record `FeatureAtomics32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureAtomics32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 573-575
```tablegen
//===----------------------------------------------------------------------===//
// ARM architecture class
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 577-579
```tablegen
// A-series ISA
def FeatureAClass : SubtargetFeature<"aclass", "ARMProcClass", "AClass",
                                     "Is application profile ('A' series)">;
```
- EN: Defines TableGen record `FeatureAClass` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureAClass`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 581-583
```tablegen
// R-series ISA
def FeatureRClass : SubtargetFeature<"rclass", "ARMProcClass", "RClass",
                                     "Is realtime profile ('R' series)">;
```
- EN: Defines TableGen record `FeatureRClass` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureRClass`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 585-587
```tablegen
// M-series ISA
def FeatureMClass : SubtargetFeature<"mclass", "ARMProcClass", "MClass",
                                     "Is microcontroller profile ('M' series)">;
```
- EN: Defines TableGen record `FeatureMClass` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureMClass`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 589-591
```tablegen
// True if Thumb2 instructions are supported.
def FeatureThumb2 : SubtargetFeature<"thumb2", "HasThumb2", "true",
                                     "Enable Thumb2 instructions">;
```
- EN: Defines TableGen record `FeatureThumb2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureThumb2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 593-595
```tablegen
// True if subtarget does not support ARM mode execution.
def FeatureNoARM  : SubtargetFeature<"noarm", "NoARM", "true",
                                     "Does not support ARM mode execution">;
```
- EN: Defines TableGen record `FeatureNoARM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureNoARM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 597-600
```tablegen
//===----------------------------------------------------------------------===//
// ARM ISAa.
//
// Specify whether target support specific ARM ISA variants.
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 602-603
```tablegen
def HasV4TOps   : SubtargetFeature<"v4t", "HasV4TOps", "true",
                                   "Support ARM v4T instructions">;
```
- EN: Defines TableGen record `HasV4TOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV4TOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 605-607
```tablegen
def HasV5TOps   : SubtargetFeature<"v5t", "HasV5TOps", "true",
                                   "Support ARM v5T instructions",
                                   [HasV4TOps]>;
```
- EN: Defines TableGen record `HasV5TOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV5TOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 609-612
```tablegen
def HasV5TEOps  : SubtargetFeature<"v5te", "HasV5TEOps", "true",
                                   "Support ARM v5TE, v5TEj, and "
                                   "v5TExp instructions",
                                   [HasV5TOps]>;
```
- EN: Defines TableGen record `HasV5TEOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV5TEOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 614-616
```tablegen
def HasV6Ops    : SubtargetFeature<"v6", "HasV6Ops", "true",
                                   "Support ARM v6 instructions",
                                   [HasV5TEOps]>;
```
- EN: Defines TableGen record `HasV6Ops` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV6Ops`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 618-620
```tablegen
def HasV6MOps   : SubtargetFeature<"v6m", "HasV6MOps", "true",
                                   "Support ARM v6M instructions",
                                   [HasV6Ops]>;
```
- EN: Defines TableGen record `HasV6MOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV6MOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 622-624
```tablegen
def HasV8MBaselineOps : SubtargetFeature<"v8m", "HasV8MBaselineOps", "true",
                                         "Support ARM v8M Baseline instructions",
                                         [HasV6MOps]>;
```
- EN: Defines TableGen record `HasV8MBaselineOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV8MBaselineOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 626-628
```tablegen
def HasV6KOps   : SubtargetFeature<"v6k", "HasV6KOps", "true",
                                   "Support ARM v6k instructions",
                                   [HasV6Ops]>;
```
- EN: Defines TableGen record `HasV6KOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV6KOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 630-632
```tablegen
def HasV6T2Ops  : SubtargetFeature<"v6t2", "HasV6T2Ops", "true",
                                   "Support ARM v6t2 instructions",
                                   [HasV8MBaselineOps, HasV6KOps, FeatureThumb2]>;
```
- EN: Defines TableGen record `HasV6T2Ops` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV6T2Ops`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 634-636
```tablegen
def HasV7Ops    : SubtargetFeature<"v7", "HasV7Ops", "true",
                                   "Support ARM v7 instructions",
                                   [HasV6T2Ops, FeatureV7Clrex]>;
```
- EN: Defines TableGen record `HasV7Ops` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV7Ops`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 638-641
```tablegen
def HasV8MMainlineOps :
                  SubtargetFeature<"v8m.main", "HasV8MMainlineOps", "true",
                                   "Support ARM v8M Mainline instructions",
                                   [HasV7Ops]>;
```
- EN: Defines TableGen record `HasV8MMainlineOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV8MMainlineOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 643-645
```tablegen
def HasV8Ops    : SubtargetFeature<"v8", "HasV8Ops", "true",
                                   "Support ARM v8 instructions",
                                   [HasV7Ops, FeaturePerfMon, FeatureAcquireRelease]>;
```
- EN: Defines TableGen record `HasV8Ops` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV8Ops`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 647-649
```tablegen
def HasV8_1aOps : SubtargetFeature<"v8.1a", "HasV8_1aOps", "true",
                                   "Support ARM v8.1a instructions",
                                   [HasV8Ops]>;
```
- EN: Defines TableGen record `HasV8_1aOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV8_1aOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 651-653
```tablegen
def HasV8_2aOps : SubtargetFeature<"v8.2a", "HasV8_2aOps", "true",
                                   "Support ARM v8.2a instructions",
                                   [HasV8_1aOps]>;
```
- EN: Defines TableGen record `HasV8_2aOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV8_2aOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 655-657
```tablegen
def HasV8_3aOps   : SubtargetFeature<"v8.3a", "HasV8_3aOps", "true",
                                   "Support ARM v8.3a instructions",
                                   [HasV8_2aOps]>;
```
- EN: Defines TableGen record `HasV8_3aOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV8_3aOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 659-661
```tablegen
def HasV8_4aOps   : SubtargetFeature<"v8.4a", "HasV8_4aOps", "true",
                                   "Support ARM v8.4a instructions",
                                   [HasV8_3aOps, FeatureDotProd]>;
```
- EN: Defines TableGen record `HasV8_4aOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV8_4aOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 663-665
```tablegen
def HasV8_5aOps   : SubtargetFeature<"v8.5a", "HasV8_5aOps", "true",
                                   "Support ARM v8.5a instructions",
                                   [HasV8_4aOps, FeatureSB]>;
```
- EN: Defines TableGen record `HasV8_5aOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV8_5aOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 667-670
```tablegen
def HasV8_6aOps   : SubtargetFeature<"v8.6a", "HasV8_6aOps", "true",
                                   "Support ARM v8.6a instructions",
                                   [HasV8_5aOps, FeatureBF16,
                                    FeatureMatMulInt8]>;
```
- EN: Defines TableGen record `HasV8_6aOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV8_6aOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 672-674
```tablegen
def HasV8_7aOps   : SubtargetFeature<"v8.7a", "HasV8_7aOps", "true",
                                   "Support ARM v8.7a instructions",
                                   [HasV8_6aOps]>;
```
- EN: Defines TableGen record `HasV8_7aOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV8_7aOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 676-678
```tablegen
def HasV8_8aOps   : SubtargetFeature<"v8.8a", "HasV8_8aOps", "true",
                                   "Support ARM v8.8a instructions",
                                   [HasV8_7aOps]>;
```
- EN: Defines TableGen record `HasV8_8aOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV8_8aOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 680-682
```tablegen
def HasV8_9aOps   : SubtargetFeature<"v8.9a", "HasV8_9aOps", "true",
                                   "Support ARM v8.9a instructions",
                                   [HasV8_8aOps, FeatureCLRBHB]>;
```
- EN: Defines TableGen record `HasV8_9aOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV8_9aOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 684-686
```tablegen
def HasV9_0aOps   : SubtargetFeature<"v9a", "HasV9_0aOps", "true",
                                   "Support ARM v9a instructions",
                                   [HasV8_5aOps]>;
```
- EN: Defines TableGen record `HasV9_0aOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV9_0aOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 688-690
```tablegen
def HasV9_1aOps   : SubtargetFeature<"v9.1a", "HasV9_1aOps", "true",
                                   "Support ARM v9.1a instructions",
                                   [HasV8_6aOps, HasV9_0aOps]>;
```
- EN: Defines TableGen record `HasV9_1aOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV9_1aOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 692-694
```tablegen
def HasV9_2aOps   : SubtargetFeature<"v9.2a", "HasV9_2aOps", "true",
                                   "Support ARM v9.2a instructions",
                                   [HasV8_7aOps, HasV9_1aOps]>;
```
- EN: Defines TableGen record `HasV9_2aOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV9_2aOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 696-698
```tablegen
def HasV9_3aOps   : SubtargetFeature<"v9.3a", "HasV9_3aOps", "true",
                                   "Support ARM v9.3a instructions",
                                   [HasV8_8aOps, HasV9_2aOps]>;
```
- EN: Defines TableGen record `HasV9_3aOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV9_3aOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 700-702
```tablegen
def HasV9_4aOps   : SubtargetFeature<"v9.4a", "HasV9_4aOps", "true",
                                   "Support ARM v9.4a instructions",
                                   [HasV8_9aOps, HasV9_3aOps]>;
```
- EN: Defines TableGen record `HasV9_4aOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV9_4aOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 704-708
```tablegen
// Armv9.5-A is a v9-only architecture. From v9.5-A onwards there's no mapping
// to an equivalent v8.x version.
def HasV9_5aOps   : SubtargetFeature<"v9.5a", "HasV9_5aOps", "true",
                                   "Support ARM v9.5a instructions",
                                   [HasV9_4aOps]>;
```
- EN: Defines TableGen record `HasV9_5aOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV9_5aOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 710-713
```tablegen
// Armv9.6-A is a v9-only architecture.
def HasV9_6aOps   : SubtargetFeature<"v9.6a", "HasV9_6aOps", "true",
                                   "Support ARM v9.6a instructions",
                                   [HasV9_5aOps]>;
```
- EN: Defines TableGen record `HasV9_6aOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV9_6aOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 715-718
```tablegen
// Armv9.7-A is a v9-only architecture.
def HasV9_7aOps   : SubtargetFeature<"v9.7a", "HasV9_7aOps", "true",
                                   "Support ARM v9.7a instructions",
                                   [HasV9_6aOps]>;
```
- EN: Defines TableGen record `HasV9_7aOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV9_7aOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 720-731
```tablegen
def HasV8_1MMainlineOps : SubtargetFeature<
               "v8.1m.main", "HasV8_1MMainlineOps", "true",
               "Support ARM v8-1M Mainline instructions",
               [HasV8MMainlineOps]>;
def HasMVEIntegerOps : SubtargetFeature<
               "mve", "HasMVEIntegerOps", "true",
               "Support M-Class Vector Extension with integer ops",
               [HasV8_1MMainlineOps, FeatureDSP, FeatureFPRegs16, FeatureFPRegs64]>;
def HasMVEFloatOps : SubtargetFeature<
               "mve.fp", "HasMVEFloatOps", "true",
               "Support M-Class Vector Extension with integer and floating ops",
               [HasMVEIntegerOps, FeatureFPARMv8_D16_SP, FeatureFullFP16]>;
```
- EN: Defines TableGen record `HasV8_1MMainlineOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasV8_1MMainlineOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 733-735
```tablegen
def HasCDEOps : SubtargetFeature<"cde", "HasCDEOps", "true",
                                 "Support CDE instructions",
                                 [HasV8MMainlineOps]>;
```
- EN: Defines TableGen record `HasCDEOps` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasCDEOps`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 737-741
```tablegen
foreach i = {0-7} in
    def FeatureCoprocCDE#i : SubtargetFeature<"cdecp"#i,
                                              "CoprocCDE["#i#"]", "true",
                                              "Coprocessor "#i#" ISA is CDEv1",
                                              [HasCDEOps]>;
```
- EN: Defines TableGen record `FeatureCoprocCDE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureCoprocCDE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 743-745
```tablegen
//===----------------------------------------------------------------------===//
// Control codegen mitigation against Straight Line Speculation vulnerability.
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 747-759
```tablegen
/// Harden against Straight Line Speculation for Returns and Indirect Branches.
def FeatureHardenSlsRetBr : SubtargetFeature<"harden-sls-retbr",
  "HardenSlsRetBr", "true",
  "Harden against straight line speculation across RETurn and BranchRegister "
  "instructions">;
/// Harden against Straight Line Speculation for indirect calls.
def FeatureHardenSlsBlr : SubtargetFeature<"harden-sls-blr",
  "HardenSlsBlr", "true",
  "Harden against straight line speculation across indirect calls">;
/// Generate thunk code for SLS mitigation in the normal text section.
def FeatureHardenSlsNoComdat : SubtargetFeature<"harden-sls-nocomdat",
  "HardenSlsNoComdat", "true",
  "Generate thunk code for SLS mitigation in the normal text section">;
```
- EN: Defines TableGen record `FeatureHardenSlsRetBr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FeatureHardenSlsRetBr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 761-777
```tablegen
//===----------------------------------------------------------------------===//
// Endianness of instruction encodings in memory.
//
// In the current Arm architecture, this is usually little-endian regardless of
// data endianness. But before Armv7 it was typical for instruction endianness
// to match data endianness, so that a big-endian system was consistently big-
// endian. And Armv7-R can be configured to use big-endian instructions.
//
// Additionally, even when targeting Armv7-A, big-endian instructions can be
// found in relocatable object files, because the Arm ABI specifies that the
// linker byte-reverses them depending on the target architecture.
//
// So we have a feature here to indicate that instructions are stored big-
// endian, which you can set when instantiating an MCDisassembler.
def ModeBigEndianInstructions : SubtargetFeature<"big-endian-instructions",
    "BigEndianInstructions", "true",
     "Expect instructions to be stored big-endian.">;
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

## Key Concepts / 关键概念

- EN: Primary role: feature flags and capability predicates.
  - CN: 核心职责：特性标志与能力判定。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `(none)`.
  - CN: TableGen 包含项：`(none)`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
