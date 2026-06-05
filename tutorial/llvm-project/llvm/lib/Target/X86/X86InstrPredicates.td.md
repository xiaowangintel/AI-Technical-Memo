# X86InstrPredicates.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86InstrPredicates.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines instruction definitions and metadata for the core X86 backend using LLVM TableGen DSL. / 使用 LLVM TableGen DSL 为X86 后端核心定义指令定义与元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```tablegen
//===---X86InstrPredicates.td - X86 Predicate Definitions --*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

def TruePredicate : Predicate<"true">;

// Intel x86 instructions have three separate encoding spaces: legacy, VEX, and
// EVEX. Not all X86 instructions are extended for EGPR. The following is an
// overview of which instructions are extended and how we implement them.
//
// * Legacy space
//   All instructions in legacy maps 0 and 1 that have explicit GPR or memory
//   operands can use the REX2 prefix to access the EGPR, except XSAVE*/XRSTOR.
//
// * EVEX space
//   All instructions in the EVEX space can access the EGPR in their
//   register/memory operands.
//
// For the above intructions, the only difference in encoding is reflected in
// the REX2/EVEX prefix when EGPR is used, i.e. the opcode and opcode name are
// unchanged. We don’t add new entries in TD, and instead we extend GPR with
// R16-R31 and make them allocatable only when the feature EGPR is available.
//
// Besides, some instructions in legacy space with map 2/3 and VEX space are
// promoted into EVEX space. Encoding space changes after the promotion, opcode
// and opcode map may change too sometimes. For these instructions, we add new
```
**EN:** This section uses LLVM TableGen DSL to describe file header, licensing, and opening context for the core X86 backend. Key symbols include TruePredicate. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的文件头、许可证与开场上下文。关键符号包括 TruePredicate。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 31-60: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
// entries in TD to avoid overcomplicating the assembler and disassembler.
//
// HasEGPR is for the new entries and NoEGPR is for the entries before
// promotion, so that the promoted variant can be selected first to benefit RA.
def HasEGPR      : Predicate<"Subtarget->hasEGPR()">;
def NoEGPR       : Predicate<"!Subtarget->hasEGPR()">;

// APX extends some instructions with a new form that has an extra register
// operand called a new data destination (NDD). In such forms, NDD is the new
// destination register receiving the result of the computation and all other
// operands (including the original destination operand) become read-only source
// operands.
//
// HasNDD is for the new NDD entries and NoNDD is for the legacy 2-address
// entries, so that the NDD variant can be selected first to benefit RA.
def HasNDD       : Predicate<"Subtarget->hasNDD()">;
def NoNDD        : Predicate<"!Subtarget->hasNDD()">;
def HasZU        : Predicate<"Subtarget->hasZU()">;
def PreferLegacySetCC  : Predicate<"!Subtarget->hasZU() || "
                                   "Subtarget->preferLegacySetCC()">;
def PreferNoLegacySetCC : Predicate<"Subtarget->hasZU() && "
                                    "!Subtarget->preferLegacySetCC()">;
def HasNDDI      : Predicate<"Subtarget->hasNDD() && Subtarget->hasNDDI()">;
def NoNDDI       : Predicate<"!Subtarget->hasNDD() || !Subtarget->hasNDDI()">;
def HasCF        : Predicate<"Subtarget->hasCF()">;
def HasJMPABS    : Predicate<"Subtarget->hasJMPABS()">;
def HasCMOV      : Predicate<"Subtarget->canUseCMOV()">;
def NoCMOV       : Predicate<"!Subtarget->canUseCMOV()">;
def HasNOPL      : Predicate<"Subtarget->hasNOPL()">;
def HasMMX       : Predicate<"Subtarget->hasMMX()">;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include HasEGPR, NoEGPR, HasNDD, NoNDD, HasZU, PreferLegacySetCC. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 HasEGPR, NoEGPR, HasNDD, NoNDD, HasZU, PreferLegacySetCC。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 61-90: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
def HasSSE1      : Predicate<"Subtarget->hasSSE1()">;
def UseSSE1      : Predicate<"Subtarget->hasSSE1() && !Subtarget->hasAVX()">;
def HasSSE2      : Predicate<"Subtarget->hasSSE2()">;
def UseSSE2      : Predicate<"Subtarget->hasSSE2() && !Subtarget->hasAVX()">;
def HasSSE3      : Predicate<"Subtarget->hasSSE3()">;
def UseSSE3      : Predicate<"Subtarget->hasSSE3() && !Subtarget->hasAVX()">;
def HasSSSE3     : Predicate<"Subtarget->hasSSSE3()">;
def UseSSSE3     : Predicate<"Subtarget->hasSSSE3() && !Subtarget->hasAVX()">;
def HasSSE41     : Predicate<"Subtarget->hasSSE41()">;
def NoSSE41      : Predicate<"!Subtarget->hasSSE41()">;
def UseSSE41     : Predicate<"Subtarget->hasSSE41() && !Subtarget->hasAVX()">;
def HasSSE42     : Predicate<"Subtarget->hasSSE42()">;
def UseSSE42     : Predicate<"Subtarget->hasSSE42() && !Subtarget->hasAVX()">;
def HasSSE4A     : Predicate<"Subtarget->hasSSE4A()">;
def NoAVX        : Predicate<"!Subtarget->hasAVX()">;
def HasAVX       : Predicate<"Subtarget->hasAVX()">;
def HasAVX2      : Predicate<"Subtarget->hasAVX2()">;
def HasAVX1Only  : Predicate<"Subtarget->hasAVX() && !Subtarget->hasAVX2()">;
def HasAVX10_1   : Predicate<"Subtarget->hasAVX10_1()">;
def HasAVX10_2   : Predicate<"Subtarget->hasAVX10_2()">;
def NoAVX10_2    : Predicate<"!Subtarget->hasAVX10_2()">;
def HasAVX512    : Predicate<"Subtarget->hasAVX512()">;
def UseAVX       : Predicate<"Subtarget->hasAVX() && !Subtarget->hasAVX512()">;
def UseAVX2      : Predicate<"Subtarget->hasAVX2() && !Subtarget->hasAVX512()">;
def NoAVX512     : Predicate<"!Subtarget->hasAVX512()">;
def HasCDI       : Predicate<"Subtarget->hasCDI()">;
def HasVPOPCNTDQ : Predicate<"Subtarget->hasVPOPCNTDQ()">;
def HasDQI       : Predicate<"Subtarget->hasDQI()">;
def NoDQI        : Predicate<"!Subtarget->hasDQI()">;
def HasBWI       : Predicate<"Subtarget->hasBWI()">;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include HasSSE1, UseSSE1, HasSSE2, UseSSE2, HasSSE3, UseSSE3. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 HasSSE1, UseSSE1, HasSSE2, UseSSE2, HasSSE3, UseSSE3。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 91-120: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
def NoBWI        : Predicate<"!Subtarget->hasBWI()">;
def HasVLX       : Predicate<"Subtarget->hasVLX()">;
def NoVLX        : Predicate<"!Subtarget->hasVLX()">;
def NoVLX_Or_NoBWI : Predicate<"!Subtarget->hasVLX() || !Subtarget->hasBWI()">;
def NoVLX_Or_NoDQI : Predicate<"!Subtarget->hasVLX() || !Subtarget->hasDQI()">;
def HasPKU       : Predicate<"Subtarget->hasPKU()">;
def HasVNNI      : Predicate<"Subtarget->hasVNNI()">;
def HasVP2INTERSECT : Predicate<"Subtarget->hasVP2INTERSECT()">;
def HasBF16      : Predicate<"Subtarget->hasBF16()">;
def HasFP16      : Predicate<"Subtarget->hasFP16()">;
def HasAVXVNNIINT16 : Predicate<"Subtarget->hasAVXVNNIINT16()">;
def HasAVXVNNIINT8 : Predicate<"Subtarget->hasAVXVNNIINT8()">;
def HasAVXVNNI : Predicate <"Subtarget->hasAVXVNNI()">;
def NoVLX_Or_NoVNNI : Predicate<"!Subtarget->hasVLX() || !Subtarget->hasVNNI()">;

def HasBITALG    : Predicate<"Subtarget->hasBITALG()">;
def HasPOPCNT    : Predicate<"Subtarget->hasPOPCNT()">;
def HasAES       : Predicate<"Subtarget->hasAES()">;
def HasVAES      : Predicate<"Subtarget->hasVAES()">;
def NoVLX_Or_NoVAES : Predicate<"!Subtarget->hasVLX() || !Subtarget->hasVAES()">;
def HasFXSR      : Predicate<"Subtarget->hasFXSR()">;
def HasX87       : Predicate<"Subtarget->hasX87()">;
def HasXSAVE     : Predicate<"Subtarget->hasXSAVE()">;
def HasXSAVEOPT  : Predicate<"Subtarget->hasXSAVEOPT()">;
def HasXSAVEC    : Predicate<"Subtarget->hasXSAVEC()">;
def HasXSAVES    : Predicate<"Subtarget->hasXSAVES()">;
def HasPCLMUL    : Predicate<"Subtarget->hasPCLMUL()">;
def NoVLX_Or_NoVPCLMULQDQ :
                    Predicate<"!Subtarget->hasVLX() || !Subtarget->hasVPCLMULQDQ()">;
def HasVPCLMULQDQ : Predicate<"Subtarget->hasVPCLMULQDQ()">;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include NoBWI, HasVLX, NoVLX, NoVLX_Or_NoBWI, NoVLX_Or_NoDQI, HasPKU. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 NoBWI, HasVLX, NoVLX, NoVLX_Or_NoBWI, NoVLX_Or_NoDQI, HasPKU。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 121-150: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
def HasGFNI      : Predicate<"Subtarget->hasGFNI()">;
def HasFMA       : Predicate<"Subtarget->hasFMA()">;
def HasFMA4      : Predicate<"Subtarget->hasFMA4()">;
def NoFMA4       : Predicate<"!Subtarget->hasFMA4()">;
def HasXOP       : Predicate<"Subtarget->hasXOP()">;
def HasTBM       : Predicate<"Subtarget->hasTBM()">;
def NoTBM        : Predicate<"!Subtarget->hasTBM()">;
def HasLWP       : Predicate<"Subtarget->hasLWP()">;
def HasMOVBE     : Predicate<"Subtarget->hasMOVBE()">;
def NoNDD_Or_NoMOVBE     : Predicate<"!Subtarget->hasNDD() || !Subtarget->hasMOVBE()">;
def HasRDRAND    : Predicate<"Subtarget->hasRDRAND()">;
def HasF16C      : Predicate<"Subtarget->hasF16C()">;
def HasFSGSBase  : Predicate<"Subtarget->hasFSGSBase()">;
def HasLZCNT     : Predicate<"Subtarget->hasLZCNT()">;
def HasBMI       : Predicate<"Subtarget->hasBMI()">;
def HasBMI2      : Predicate<"Subtarget->hasBMI2()">;
def NoBMI2       : Predicate<"!Subtarget->hasBMI2()">;
def HasVBMI      : Predicate<"Subtarget->hasVBMI()">;
def HasVBMI2     : Predicate<"Subtarget->hasVBMI2()">;
def HasIFMA      : Predicate<"Subtarget->hasIFMA()">;
def HasAVXIFMA   : Predicate<"Subtarget->hasAVXIFMA()">;
def NoVLX_Or_NoIFMA : Predicate<"!Subtarget->hasVLX() || !Subtarget->hasIFMA()">;
def HasRTM       : Predicate<"Subtarget->hasRTM()">;
def HasSHA       : Predicate<"Subtarget->hasSHA()">;
def HasSHA512    : Predicate<"Subtarget->hasSHA512()">;
def HasSGX       : Predicate<"Subtarget->hasSGX()">;
def HasSM3       : Predicate<"Subtarget->hasSM3()">;
def HasRDSEED    : Predicate<"Subtarget->hasRDSEED()">;
def HasSSEPrefetch : Predicate<"Subtarget->hasSSEPrefetch()">;
def HasPRFCHW    : Predicate<"Subtarget->hasPRFCHW()">;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include HasGFNI, HasFMA, HasFMA4, NoFMA4, HasXOP, HasTBM. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 HasGFNI, HasFMA, HasFMA4, NoFMA4, HasXOP, HasTBM。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 151-180: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
def HasPREFETCHI : Predicate<"Subtarget->hasPREFETCHI()">;
def HasPrefetchW : Predicate<"Subtarget->hasPrefetchW()">;
def HasLAHFSAHF  : Predicate<"Subtarget->hasLAHFSAHF()">;
def HasLAHFSAHF64 : Predicate<"Subtarget->hasLAHFSAHF64()">;
def HasMWAITX    : Predicate<"Subtarget->hasMWAITX()">;
def HasCLZERO    : Predicate<"Subtarget->hasCLZERO()">;
def HasCLDEMOTE  : Predicate<"Subtarget->hasCLDEMOTE()">;
def HasMOVDIRI   : Predicate<"Subtarget->hasMOVDIRI()">;
def HasMOVDIR64B : Predicate<"Subtarget->hasMOVDIR64B()">;
def HasMOVRS     : Predicate<"Subtarget->hasMOVRS()">;
def HasPTWRITE   : Predicate<"Subtarget->hasPTWRITE()">;
def FPStackf32   : Predicate<"!Subtarget->hasSSE1()">;
def FPStackf64   : Predicate<"!Subtarget->hasSSE2()">;
def HasSHSTK     : Predicate<"Subtarget->hasSHSTK()">;
def HasSM4       : Predicate<"Subtarget->hasSM4()">;
def HasCLFLUSH   : Predicate<"Subtarget->hasCLFLUSH()">;
def HasCLFLUSHOPT : Predicate<"Subtarget->hasCLFLUSHOPT()">;
def HasCLWB      : Predicate<"Subtarget->hasCLWB()">;
def HasWBNOINVD  : Predicate<"Subtarget->hasWBNOINVD()">;
def HasRDPID     : Predicate<"Subtarget->hasRDPID()">;
def HasRDPRU     : Predicate<"Subtarget->hasRDPRU()">;
def HasWAITPKG   : Predicate<"Subtarget->hasWAITPKG()">;
def HasINVPCID   : Predicate<"Subtarget->hasINVPCID()">;
def HasCX8       : Predicate<"Subtarget->hasCX8()">;
def HasCX16      : Predicate<"Subtarget->hasCX16()">;
def HasPCONFIG   : Predicate<"Subtarget->hasPCONFIG()">;
def HasENQCMD    : Predicate<"Subtarget->hasENQCMD()">;
def HasAMXFP16   : Predicate<"Subtarget->hasAMXFP16()">;
def HasCMPCCXADD : Predicate<"Subtarget->hasCMPCCXADD()">;
def HasAVXNECONVERT : Predicate<"Subtarget->hasAVXNECONVERT()">;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include HasPREFETCHI, HasPrefetchW, HasLAHFSAHF, HasLAHFSAHF64, HasMWAITX, HasCLZERO. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 HasPREFETCHI, HasPrefetchW, HasLAHFSAHF, HasLAHFSAHF64, HasMWAITX, HasCLZERO。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 181-210: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
def HasKL        : Predicate<"Subtarget->hasKL()">;
def HasRAOINT    : Predicate<"Subtarget->hasRAOINT()">;
def HasWIDEKL    : Predicate<"Subtarget->hasWIDEKL()">;
def HasHRESET    : Predicate<"Subtarget->hasHRESET()">;
def HasSERIALIZE : Predicate<"Subtarget->hasSERIALIZE()">;
def HasTSXLDTRK  : Predicate<"Subtarget->hasTSXLDTRK()">;
def HasAMXTILE   : Predicate<"Subtarget->hasAMXTILE()">;
def HasAMXBF16   : Predicate<"Subtarget->hasAMXBF16()">;
def HasAMXINT8   : Predicate<"Subtarget->hasAMXINT8()">;
def HasAMXCOMPLEX : Predicate<"Subtarget->hasAMXCOMPLEX()">;
def HasAMXFP8    : Predicate<"Subtarget->hasAMXFP8()">;
def HasAMXMOVRS  : Predicate<"Subtarget->hasAMXMOVRS()">;
def HasAMXAVX512 : Predicate<"Subtarget->hasAMXAVX512()">;
def HasAMXTF32   : Predicate<"Subtarget->hasAMXTF32()">;
def HasUINTR     : Predicate<"Subtarget->hasUINTR()">;
def HasUSERMSR   : Predicate<"Subtarget->hasUSERMSR()">;
def HasCRC32     : Predicate<"Subtarget->hasCRC32()">;

def HasX86_64    : Predicate<"Subtarget->hasX86_64()">;
def Not64BitMode : Predicate<"!Subtarget->is64Bit()">,
                             AssemblerPredicate<(all_of (not Is64Bit)), "Not 64-bit mode">;
def In64BitMode  : Predicate<"Subtarget->is64Bit()">,
                             AssemblerPredicate<(all_of Is64Bit), "64-bit mode">;

def IsX32Mode  : Predicate<"Subtarget->getTargetTriple().isX32()">,
                            AssemblerPredicate<(all_of IsX32), "x32 ABI">;
def NotX32Mode  : Predicate<"!Subtarget->getTargetTriple().isX32()">,
                            AssemblerPredicate<(all_of (not IsX32)), "not x32 ABI">;

def IsLP64  : Predicate<"Subtarget->isTarget64BitLP64()">;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include HasKL, HasRAOINT, HasWIDEKL, HasHRESET, HasSERIALIZE, HasTSXLDTRK. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 HasKL, HasRAOINT, HasWIDEKL, HasHRESET, HasSERIALIZE, HasTSXLDTRK。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 211-240: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
def NotLP64 : Predicate<"!Subtarget->isTarget64BitLP64()">;
def In16BitMode  : Predicate<"Subtarget->is16Bit()">,
                             AssemblerPredicate<(all_of Is16Bit), "16-bit mode">;
def Not16BitMode : Predicate<"!Subtarget->is16Bit()">,
                             AssemblerPredicate<(all_of (not Is16Bit)), "Not 16-bit mode">;
def In32BitMode  : Predicate<"Subtarget->is32Bit()">,
                             AssemblerPredicate<(all_of Is32Bit), "32-bit mode">;
def IsWin64      : Predicate<"Subtarget->isTargetWin64()">;
def NotWin64     : Predicate<"!Subtarget->isTargetWin64()">;
def NotWin64WithoutFP : Predicate<"!Subtarget->isTargetWin64() ||"
                                  "Subtarget->getFrameLowering()->hasFP(*MF)"> {
  let RecomputePerFunction = 1;
}
def IsPS         : Predicate<"Subtarget->isTargetPS()">;
def NotPS        : Predicate<"!Subtarget->isTargetPS()">;
def SmallCode    : Predicate<"TM.getCodeModel() == CodeModel::Small">;
def KernelCode   : Predicate<"TM.getCodeModel() == CodeModel::Kernel">;
def NearData     : Predicate<"TM.getCodeModel() == CodeModel::Small ||"
                             "TM.getCodeModel() == CodeModel::Kernel">;
def IsNotPIC     : Predicate<"!TM.isPositionIndependent()">;

// We could compute these on a per-module basis but doing so requires accessing
// the Function object through the <Target>Subtarget and objections were raised
// to that (see post-commit review comments for r301750).
let RecomputePerFunction = 1 in {
  def OptForSize   : Predicate<"shouldOptForSize(MF)">;
  def OptForMinSize : Predicate<"MF->getFunction().hasMinSize()">;
  def OptForSpeed  : Predicate<"!shouldOptForSize(MF)">;
  def UseIncDec : Predicate<"!Subtarget->slowIncDec() || "
                            "shouldOptForSize(MF)">;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include NotLP64, In16BitMode, Not16BitMode, In32BitMode, IsWin64, NotWin64. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 NotLP64, In16BitMode, Not16BitMode, In32BitMode, IsWin64, NotWin64。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 241-270: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
  def NoSSE41_Or_OptForSize : Predicate<"shouldOptForSize(MF) || "
                                        "!Subtarget->hasSSE41()">;
  def ImportCallOptimizationEnabled : Predicate<"MF->getFunction().getParent()->getModuleFlag(\"import-call-optimization\")">;
  def ImportCallOptimizationDisabled : Predicate<"!MF->getFunction().getParent()->getModuleFlag(\"import-call-optimization\")">;

  def IsWin64CCFunc : Predicate<"Subtarget->isCallingConvWin64(MF->getFunction().getCallingConv())">;
  def IsNotWin64CCFunc : Predicate<"!Subtarget->isCallingConvWin64(MF->getFunction().getCallingConv())">;
  def IsHiPECCFunc : Predicate<"MF->getFunction().getCallingConv() == CallingConv::HiPE">;

  def IsNotHiPECCFunc : Predicate<
    "MF->getFunction().getCallingConv() != CallingConv::HiPE">;
}

def CallImmAddr  : Predicate<"Subtarget->isLegalToCallImmediateAddr()">;
def FavorMemIndirectCall  : Predicate<"!Subtarget->slowTwoMemOps() && "
                                      "!Subtarget->slowIndirectCall()">;
def HasFastMem32 : Predicate<"!Subtarget->isUnalignedMem32Slow()">;
def HasFastLZCNT : Predicate<"Subtarget->hasFastLZCNT()">;
def HasFastSHLDRotate : Predicate<"Subtarget->hasFastSHLDRotate()">;
def HasERMSB : Predicate<"Subtarget->hasERMSB()">;
def HasFSRM : Predicate<"Subtarget->hasFSRM()">;
def HasMFence    : Predicate<"Subtarget->hasMFence()">;
def HasFastDPWSSD: Predicate<"Subtarget->hasFastDPWSSD()">;
def UseIndirectThunkCalls : Predicate<"Subtarget->useIndirectThunkCalls()">;
def NotUseIndirectThunkCalls : Predicate<"!Subtarget->useIndirectThunkCalls()">;

//===----------------------------------------------------------------------===//
// HwModes
//===----------------------------------------------------------------------===//

```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include NoSSE41_Or_OptForSize, ImportCallOptimizationEnabled, ImportCallOptimizationDisabled, IsWin64CCFunc, IsNotWin64CCFunc, IsHiPECCFunc. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 NoSSE41_Or_OptForSize, ImportCallOptimizationEnabled, ImportCallOptimizationDisabled, IsWin64CCFunc, IsNotWin64CCFunc, IsHiPECCFunc。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 271-273: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
defvar X86_32 = DefaultMode;
def X86_64 : HwMode<[In64BitMode, NotX32Mode]>;
def X86_64_X32 : HwMode<[IsX32Mode]>;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include X86_64, X86_64_X32. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 X86_64, X86_64_X32。这些记录随后会展开为 X86 后端使用的生成表。

## Key Concepts / 关键概念
- Primary topic: instruction definitions and metadata. / 核心主题：指令定义与元数据。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: TruePredicate, HasEGPR, NoEGPR, HasNDD, NoNDD, HasZU. / 重要符号：TruePredicate, HasEGPR, NoEGPR, HasNDD, NoNDD, HasZU。
- This file is written in TableGen DSL and contributes generated target tables. / 该文件使用 TableGen DSL 编写，并参与生成目标描述表。

## Dependencies / 依赖关系
- Consumed by LLVM TableGen to emit generated matcher, encoding, register, or scheduling tables. / 由 LLVM TableGen 消费，以生成匹配器、编码、寄存器或调度相关表。
- Closely coupled with sibling X86 .td fragments that assemble the full target description. / 与同级 X86 .td 片段紧密协作，共同组成完整目标描述。
