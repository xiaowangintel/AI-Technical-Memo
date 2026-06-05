# AArch64.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/AArch64.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Instruction Descriptions. It is written in the TableGen DSL. / 该文件实现 AArch64 后端中的AArch64 后端逻辑。 它使用 TableGen DSL 描述目标规则。
- **Note**: Written in the TableGen DSL / 使用 TableGen DSL 编写
## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Documented TableGen section
```tablegen
//=- AArch64.td - Describe the AArch64 Target Machine --------*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// Target-independent interfaces which we are implementing.
//===----------------------------------------------------------------------===//

include "llvm/Target/Target.td"

//===----------------------------------------------------------------------===//
// Subtarget features.
//===----------------------------------------------------------------------===//
include "AArch64Features.td"
include "AArch64FMV.td"

//===----------------------------------------------------------------------===//
// Register File Description
//===----------------------------------------------------------------------===//
```
**EN:** This comment block marks a new TableGen section and explains the intent of the records that follow.  
**CN:** 该注释块标记了新的 TableGen 章节，并说明后续记录的设计意图。
### Lines 27-51: TableGen includes and shared records
```tablegen

include "AArch64RegisterInfo.td"
include "AArch64RegisterBanks.td"
include "AArch64CallingConvention.td"

//===----------------------------------------------------------------------===//
// Instruction Descriptions
//===----------------------------------------------------------------------===//

include "AArch64Schedule.td"
include "AArch64InstrInfo.td"
include "AArch64SchedPredicates.td"
include "AArch64SchedPredExynos.td"
include "AArch64SchedPredNeoverse.td"
include "AArch64Combine.td"

defm : RemapAllTargetPseudoPointerOperands<GPR64sp>;

def AArch64InstrInfo : InstrInfo;

//===----------------------------------------------------------------------===//
// Named operands for MRS/MSR/TLBI/...
//===----------------------------------------------------------------------===//

include "AArch64SystemOperands.td"
```
**EN:** This TableGen block imports shared records so the file can build larger AArch64 descriptions on top of them.  
**CN:** 该 TableGen 代码块导入共享记录，使本文件能够在其之上继续构建更完整的 AArch64 描述。
### Lines 52-77: Documented TableGen section
```tablegen

//===----------------------------------------------------------------------===//
// AArch64 Processors supported.
//

//===----------------------------------------------------------------------===//
// Unsupported features to disable for scheduling models
//===----------------------------------------------------------------------===//

class AArch64Unsupported { list<Predicate> F; }

let F = [HasSVE2p1, HasSVE2p1_or_SME2, HasSVE2p1_or_StreamingSME2, HasSVE2p1_or_SME2p1] in
def SVE2p1Unsupported : AArch64Unsupported;

def SVE2Unsupported : AArch64Unsupported {
  let F = !listconcat([HasSVE2, HasSVE2_or_SME, HasNonStreamingSVE2_or_SME2, HasSSVE_FP8FMA, HasSMEF8F16,
                       HasSSVE_FP8DOT2, HasSSVE_FP8DOT4,
                       HasSMEF8F32, HasSVEAES, HasSVESHA3, HasSVESM4, HasSVEBitPerm,
                       HasSVEB16B16],
                       SVE2p1Unsupported.F);
}

def SVEUnsupported : AArch64Unsupported {
  let F = !listconcat([HasSVE, HasSVE_or_SME],
                      SVE2Unsupported.F);
}
```
**EN:** This comment block marks a new TableGen section and explains the intent of the records that follow.  
**CN:** 该注释块标记了新的 TableGen 章节，并说明后续记录的设计意图。
### Lines 78-103: TableGen definition SME2p3Unsupported
```tablegen

def SME2p3Unsupported : AArch64Unsupported {
  let F = [HasSVE2p3_or_SME2p3, HasSVE_B16MM];
}

def SME2p2Unsupported : AArch64Unsupported {
  let F = !listconcat([HasSME2p2, HasSVE2p2_or_SME2p2,
           HasNonStreamingSVE_or_SME2p2,
           HasNonStreamingSVE2p2_or_SME2p2],
           SME2p3Unsupported.F);
}

def SME2p1Unsupported : AArch64Unsupported {
  let F = !listconcat([HasSME2p1, HasSVE2p1_or_SME2p1,
                       HasSME_MOP4, HasSME_TMOP, HasNonStreamingSVE_or_SSVE_FEXPA, 
                       HasNonStreamingSVE_or_SSVE_BitPerm],
                       SME2p2Unsupported.F);
}

def SME2Unsupported : AArch64Unsupported {
  let F = !listconcat([HasSME2, HasNonStreamingSVE2_or_SME2, HasSVE2p1_or_SME2, HasSSVE_FP8FMA,
                      HasSSVE_FP8DOT2, HasSSVE_FP8DOT4,
                      HasSMEF8F16, HasSMEF8F32, HasSMEF16F16_or_SMEF8F16, HasSMEB16B16,
                      HasNonStreamingSVE_or_SSVE_AES, HasSVE2p1_or_StreamingSME2],
                      SME2p1Unsupported.F);
}
```
**EN:** This definition materializes SME2p3Unsupported as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 SME2p3Unsupported 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 104-130: TableGen definition SMEUnsupported
```tablegen

def SMEUnsupported : AArch64Unsupported {
  let F = !listconcat([HasSME, HasSMEI16I64, HasSMEF16F16, HasSMEF64F64, HasSMEFA64],
                      SME2Unsupported.F);
}

def MTEUnsupported : AArch64Unsupported {
  let F = [HasMTE];
}

let F = [HasPAuth, HasPAuthLR] in
def PAUnsupported : AArch64Unsupported;

include "AArch64SchedA320.td"
include "AArch64SchedA53.td"
include "AArch64SchedA55.td"
include "AArch64SchedA510.td"
include "AArch64SchedA57.td"
include "AArch64SchedC1Nano.td"
include "AArch64SchedC1Ultra.td"
include "AArch64SchedC1Premium.td"
include "AArch64SchedCyclone.td"
include "AArch64SchedFalkor.td"
include "AArch64SchedKryo.td"
include "AArch64SchedExynosM3.td"
include "AArch64SchedExynosM4.td"
include "AArch64SchedExynosM5.td"
```
**EN:** This definition materializes SMEUnsupported as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 SMEUnsupported 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 131-152: TableGen includes and shared records
```tablegen
include "AArch64SchedThunderX.td"
include "AArch64SchedThunderX2T99.td"
include "AArch64SchedA64FX.td"
include "AArch64SchedThunderX3T110.td"
include "AArch64SchedTSV110.td"
include "AArch64SchedAmpere1.td"
include "AArch64SchedAmpere1B.td"
include "AArch64SchedNeoverseN1.td"
include "AArch64SchedNeoverseN2.td"
include "AArch64SchedNeoverseN3.td"
include "AArch64SchedNeoverseV1.td"
include "AArch64SchedNeoverseV2.td"
include "AArch64SchedNeoverseV3.td"
include "AArch64SchedNeoverseV3AE.td"
include "AArch64SchedOlympus.td"
include "AArch64SchedOryon.td"

include "AArch64Processors.td"

//===----------------------------------------------------------------------===//
// Assembly parser
//===----------------------------------------------------------------------===//
```
**EN:** This TableGen block imports shared records so the file can build larger AArch64 descriptions on top of them.  
**CN:** 该 TableGen 代码块导入共享记录，使本文件能够在其之上继续构建更完整的 AArch64 描述。
### Lines 153-178: TableGen definition GenericAsmParserVariant
```tablegen

def GenericAsmParserVariant : AsmParserVariant {
  int Variant = 0;
  string Name = "generic";
  string BreakCharacters = ".";
  string TokenizingCharacters = "[]*!/";
}

def AppleAsmParserVariant : AsmParserVariant {
  int Variant = 1;
  string Name = "apple-neon";
  string BreakCharacters = ".";
  string TokenizingCharacters = "[]*!/";
}

//===----------------------------------------------------------------------===//
// Assembly printer
//===----------------------------------------------------------------------===//
// AArch64 Uses the MC printer for asm output, so make sure the TableGen
// AsmWriter bits get associated with the correct class.
def GenericAsmWriter : AsmWriter {
  string AsmWriterClassName  = "InstPrinter";
  int PassSubtarget = 1;
  int Variant = 0;
  bit isMCAsmWriter = 1;
}
```
**EN:** This definition materializes GenericAsmParserVariant as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 GenericAsmParserVariant 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
### Lines 179-202: TableGen definition AppleAsmWriter
```tablegen

def AppleAsmWriter : AsmWriter {
  let AsmWriterClassName = "AppleInstPrinter";
  int PassSubtarget = 1;
  int Variant = 1;
  int isMCAsmWriter = 1;
}

//===----------------------------------------------------------------------===//
// Target Declaration
//===----------------------------------------------------------------------===//

def AArch64 : Target {
  let InstructionSet = AArch64InstrInfo;
  let AssemblyParserVariants = [GenericAsmParserVariant, AppleAsmParserVariant];
  let AssemblyWriters = [GenericAsmWriter, AppleAsmWriter];
  let AllowRegisterRenaming = 1;
}

//===----------------------------------------------------------------------===//
// Pfm Counters
//===----------------------------------------------------------------------===//

include "AArch64PfmCounters.td"
```
**EN:** This definition materializes AppleAsmWriter as a concrete backend record that TableGen can later turn into generated matchers, encoders, or metadata.  
**CN:** 该定义将 AppleAsmWriter 实例化为具体后端记录，后续 TableGen 可据此生成匹配器、编码器或元数据。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
- **EN:** Declarative backend description in TableGen DSL **CN:** 使用 TableGen DSL 的声明式后端描述
## Dependencies / 依赖关系
- **EN:** TableGen includes: llvm/Target/Target.td, AArch64Features.td, AArch64FMV.td, AArch64RegisterInfo.td, AArch64RegisterBanks.td, AArch64CallingConvention.td, AArch64Schedule.td, AArch64InstrInfo.td, AArch64SchedPredicates.td, AArch64SchedPredExynos.td, AArch64SchedPredNeoverse.td, AArch64Combine.td ... **CN:** TableGen 包含文件：llvm/Target/Target.td, AArch64Features.td, AArch64FMV.td, AArch64RegisterInfo.td, AArch64RegisterBanks.td, AArch64CallingConvention.td, AArch64Schedule.td, AArch64InstrInfo.td, AArch64SchedPredicates.td, AArch64SchedPredExynos.td, AArch64SchedPredNeoverse.td, AArch64Combine.td ...
- **EN:** Consumed by LLVM TableGen to produce generated matchers, encoders, register info, or scheduling tables. **CN:** 该文件会被 LLVM TableGen 消费，用于生成匹配器、编码器、寄存器信息或调度表。
- **EN:** Closely connected with neighboring AArch64 backend components responsible for AArch64 backend logic. **CN:** 与周边负责AArch64 后端逻辑的 AArch64 后端组件紧密协作。
