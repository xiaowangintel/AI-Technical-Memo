# ARMScheduleSwift.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMScheduleSwift.td`
- Repository: `llvm-project`
- Purpose (EN): This file defines the itinerary class data for the Swift processor.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 ARM 后端中的 `ARMScheduleSwift`，涵盖目标描述、指令记录以及代码生成元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//=- ARMScheduleSwift.td - Swift Scheduling Definitions -*- tablegen -*----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the itinerary class data for the Swift processor..
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 13-15
```tablegen
// ===---------------------------------------------------------------------===//
// This section contains legacy support for itineraries. This is
// required until SD and PostRA schedulers are replaced by MachineScheduler.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 17-19
```tablegen
def SW_DIS0 : FuncUnit;
def SW_DIS1 : FuncUnit;
def SW_DIS2 : FuncUnit;
```
- EN: Defines TableGen record `SW_DIS0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SW_DIS0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 21-25
```tablegen
def SW_ALU0 : FuncUnit;
def SW_ALU1 : FuncUnit;
def SW_LS   : FuncUnit;
def SW_IDIV : FuncUnit;
def SW_FDIV : FuncUnit;
```
- EN: Defines TableGen record `SW_ALU0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SW_ALU0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 27-37
```tablegen
// FIXME: Need bypasses.
// FIXME: Model the multiple stages of IIC_iMOVix2, IIC_iMOVix2addpc, and
//        IIC_iMOVix2ld better.
// FIXME: Model the special immediate shifts that are not microcoded.
// FIXME: Do we need to model the fact that uses of r15 in a micro-op force it
//        to issue on pipe 1?
// FIXME: Model the pipelined behavior of CMP / TST instructions.
// FIXME: Better model the microcode stages of multiply instructions, especially
//        conditional variants.
// FIXME: Add preload instruction when it is documented.
// FIXME: Model non-pipelined nature of FP div / sqrt unit.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 39-45
```tablegen
// Swift machine model for scheduling and other instruction cost heuristics.
def SwiftModel : SchedMachineModel {
  let IssueWidth = 3; // 3 micro-ops are dispatched per cycle.
  let MicroOpBufferSize = 45; // Based on NEON renamed registers.
  let LoadLatency = 3;
  let MispredictPenalty = 14; // A branch direction mispredict.
  let CompleteModel = 0;      // FIXME: Remove if all instructions are covered.
```
- EN: Defines TableGen record `SwiftModel` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftModel`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 47-49
```tablegen
  // FIXME: Remove when all errors have been fixed.
  let FullInstRWOverlapCheck = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 51-52
```tablegen
// Swift predicates.
def IsFastImmShiftSwiftPred : SchedPredicate<[{TII->isSwiftFastImmShift(MI)}]>;
```
- EN: Defines TableGen record `IsFastImmShiftSwiftPred` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IsFastImmShiftSwiftPred`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 54-61
```tablegen
// Swift resource mapping.
let SchedModel = SwiftModel in {
  // Processor resources.
  def SwiftUnitP01 : ProcResource<2>; // ALU unit.
  def SwiftUnitP0 : ProcResource<1> { let Super = SwiftUnitP01; } // Mul unit.
  def SwiftUnitP1 : ProcResource<1> { let Super = SwiftUnitP01; } // Br unit.
  def SwiftUnitP2 : ProcResource<1>; // LS unit.
  def SwiftUnitDiv : ProcResource<1>;
```
- EN: Defines TableGen record `SwiftUnitP01` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftUnitP01`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 63-80
```tablegen
  // Generic resource requirements.
  def SwiftWriteP0OneCycle : SchedWriteRes<[SwiftUnitP0]>;
  def SwiftWriteP0TwoCycle : SchedWriteRes<[SwiftUnitP0]> { let Latency = 2; }
  def SwiftWriteP0FourCycle : SchedWriteRes<[SwiftUnitP0]> { let Latency = 4; }
  def SwiftWriteP0SixCycle : SchedWriteRes<[SwiftUnitP0]> { let Latency = 6; }
  def SwiftWriteP0P1FourCycle : SchedWriteRes<[SwiftUnitP0, SwiftUnitP1]> {
    let Latency = 4;
  }
  def SwiftWriteP0P1SixCycle : SchedWriteRes<[SwiftUnitP0, SwiftUnitP1]> {
    let Latency = 6;
  }
  def SwiftWriteP01OneCycle : SchedWriteRes<[SwiftUnitP01]>;
  def SwiftWriteP1TwoCycle : SchedWriteRes<[SwiftUnitP1]> { let Latency = 2; }
  def SwiftWriteP1FourCycle : SchedWriteRes<[SwiftUnitP1]> { let Latency = 4; }
  def SwiftWriteP1SixCycle : SchedWriteRes<[SwiftUnitP1]> { let Latency = 6; }
  def SwiftWriteP1EightCycle : SchedWriteRes<[SwiftUnitP1]> { let Latency = 8; }
  def SwiftWriteP1TwelveCyc : SchedWriteRes<[SwiftUnitP1]> { let Latency = 12; }
  def SwiftWriteP01OneCycle2x : WriteSequence<[SwiftWriteP01OneCycle], 2>;
```
- EN: Defines TableGen record `SwiftWriteP0OneCycle` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftWriteP0OneCycle`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 81-98
```tablegen
  def SwiftWriteP01OneCycle3x : WriteSequence<[SwiftWriteP01OneCycle], 3>;
  def SwiftWriteP01TwoCycle : SchedWriteRes<[SwiftUnitP01]> { let Latency = 2; }
  def SwiftWriteP01ThreeCycleTwoUops : SchedWriteRes<[SwiftUnitP01,
                                                      SwiftUnitP01]> {
    let Latency = 3;
    let NumMicroOps = 2;
  }
  def SwiftWriteP0ThreeCycleThreeUops : SchedWriteRes<[SwiftUnitP0]> {
    let Latency = 3;
    let NumMicroOps = 3;
    let ReleaseAtCycles = [3];
  }
  // Plain load without writeback.
  def SwiftWriteP2ThreeCycle : SchedWriteRes<[SwiftUnitP2]> {
    let Latency = 3;
  }
  def SwiftWriteP2FourCycle : SchedWriteRes<[SwiftUnitP2]> {
    let Latency = 4;
```
- EN: Defines TableGen record `SwiftWriteP01OneCycle3x` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftWriteP01OneCycle3x`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 99-116
```tablegen
  }
  // A store does not write to a register.
  def SwiftWriteP2 : SchedWriteRes<[SwiftUnitP2]> {
    let Latency = 0;
  }
  foreach Num = 1-4 in {
    def SwiftWrite#Num#xP2 : WriteSequence<[SwiftWriteP2], Num>;
  }
  def SwiftWriteP01OneCycle2x_load : WriteSequence<[SwiftWriteP01OneCycle,
                                                    SwiftWriteP01OneCycle,
                                                    SwiftWriteP2ThreeCycle]>;
  // 4.2.4 Arithmetic and Logical.
  // ALU operation register shifted by immediate variant.
  def SwiftWriteALUsi : SchedWriteVariant<[
    // lsl #2, lsl #1, or lsr #1.
    SchedVar<IsFastImmShiftSwiftPred, [SwiftWriteP01TwoCycle]>,
    SchedVar<NoSchedPred,             [WriteALU]>
  ]>;
```
- EN: Defines TableGen record `SwiftWriteP2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftWriteP2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 117-134
```tablegen
  def SwiftWriteALUsr : SchedWriteVariant<[
    SchedVar<IsPredicatedPred, [SwiftWriteP01ThreeCycleTwoUops]>,
    SchedVar<NoSchedPred,      [SwiftWriteP01TwoCycle]>
  ]>;
  def SwiftWriteALUSsr : SchedWriteVariant<[
    SchedVar<IsPredicatedPred, [SwiftWriteP0ThreeCycleThreeUops]>,
    SchedVar<NoSchedPred,      [SwiftWriteP01TwoCycle]>
  ]>;
  def SwiftReadAdvanceALUsr : SchedReadVariant<[
    SchedVar<IsPredicatedPred, [SchedReadAdvance<2>]>,
    SchedVar<NoSchedPred,      [NoReadAdvance]>
  ]>;
  // ADC,ADD,NEG,RSB,RSC,SBC,SUB,ADR
  // AND,BIC,EOR,ORN,ORR
  // CLZ,RBIT,REV,REV16,REVSH,PKH
  def : WriteRes<WriteALU, [SwiftUnitP01]>;
  def : SchedAlias<WriteALUsi, SwiftWriteALUsi>;
  def : SchedAlias<WriteALUsr, SwiftWriteALUsr>;
```
- EN: Defines TableGen record `SwiftWriteALUsr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftWriteALUsr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 135-139
```tablegen
  def : SchedAlias<WriteALUSsr, SwiftWriteALUSsr>;
  def : ReadAdvance<ReadALU, 0>;
  def : SchedAlias<ReadALUsr, SwiftReadAdvanceALUsr>;
  def : SchedAlias<WriteLd, SwiftWriteP2ThreeCycle>;
  def : SchedAlias<WriteST, SwiftWriteP2>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 142-145
```tablegen
  def SwiftChooseShiftKindP01OneOrTwoCycle : SchedWriteVariant<[
    SchedVar<IsFastImmShiftSwiftPred, [SwiftWriteP01OneCycle]>,
    SchedVar<NoSchedPred,             [SwiftWriteP01TwoCycle]>
  ]>;
```
- EN: Defines TableGen record `SwiftChooseShiftKindP01OneOrTwoCycle` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftChooseShiftKindP01OneOrTwoCycle`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 147-150
```tablegen
  // 4.2.5 Integer comparison
  def : WriteRes<WriteCMP, [SwiftUnitP01]>;
  def : SchedAlias<WriteCMPsi, SwiftChooseShiftKindP01OneOrTwoCycle>;
  def : SchedAlias<WriteCMPsr, SwiftWriteP01TwoCycle>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 152-169
```tablegen
  // 4.2.6 Shift, Move
  // Shift
  //  ASR,LSL,ROR,RRX
  //  MOV(register-shiftedregister)  MVN(register-shiftedregister)
  // Move
  //  MOV,MVN
  //  MOVT
  // Sign/Zero extension
  def : InstRW<[SwiftWriteP01OneCycle],
               (instregex "SXTB", "SXTH", "SXTB16", "UXTB", "UXTH", "UXTB16",
                          "t2SXTB", "t2SXTH", "t2SXTB16", "t2UXTB", "t2UXTH",
                          "t2UXTB16")>;
  // Pseudo instructions.
  def : InstRW<[SwiftWriteP01OneCycle2x],
        (instregex "MOVCCi32imm", "MOVi32imm", "t2MOVCCi32imm",
                   "t2MOVi32imm")>;
  def : InstRW<[SwiftWriteP01OneCycle3x],
        (instregex "MOV_ga_pcrel", "t2MOV_ga_pcrel", "t2MOVi16_ga_pcrel")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 170-171
```tablegen
  def : InstRW<[SwiftWriteP01OneCycle2x_load],
        (instregex "MOV_ga_pcrel_ldr")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 173-173
```tablegen
  def SwiftWriteP0TwoCycleTwoUops : WriteSequence<[SwiftWriteP0OneCycle], 2>;
```
- EN: Defines TableGen record `SwiftWriteP0TwoCycleTwoUops` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftWriteP0TwoCycleTwoUops`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 175-178
```tablegen
  def SwiftPredP0OneOrTwoCycle : SchedWriteVariant<[
    SchedVar<IsPredicatedPred, [ SwiftWriteP0TwoCycleTwoUops ]>,
    SchedVar<NoSchedPred,     [ SwiftWriteP0OneCycle ]>
  ]>;
```
- EN: Defines TableGen record `SwiftPredP0OneOrTwoCycle` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftPredP0OneOrTwoCycle`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 180-182
```tablegen
  // 4.2.7 Select
  // SEL
  def : InstRW<[SwiftPredP0OneOrTwoCycle], (instregex "SEL", "t2SEL")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 184-188
```tablegen
  // 4.2.8 Bitfield
  // BFI,BFC, SBFX,UBFX
  def : InstRW< [SwiftWriteP01TwoCycle],
        (instregex "BFC", "BFI", "UBFX", "SBFX", "(t|t2)BFC", "(t|t2)BFI",
        "(t|t2)UBFX", "(t|t2)SBFX")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 190-197
```tablegen
  // 4.2.9 Saturating arithmetic
  def : InstRW< [SwiftWriteP01TwoCycle],
        (instregex "QADD", "QSUB", "QDADD", "QDSUB", "SSAT", "SSAT16", "USAT",
        "USAT16", "QADD8", "QADD16", "QSUB8", "QSUB16", "QASX", "QSAX",
        "UQADD8", "UQADD16","UQSUB8","UQSUB16","UQASX","UQSAX", "t2QADD",
        "t2QSUB", "t2QDADD", "t2QDSUB", "t2SSAT", "t2SSAT16", "t2USAT",
        "t2QADD8", "t2QADD16", "t2QSUB8", "t2QSUB16", "t2QASX", "t2QSAX",
        "t2UQADD8", "t2UQADD16","t2UQSUB8","t2UQSUB16","t2UQASX","t2UQSAX")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 199-213
```tablegen
  // 4.2.10 Parallel Arithmetic
  // Not flag setting.
  def : InstRW< [SwiftWriteALUsr],
        (instregex "SADD8", "SADD16", "SSUB8", "SSUB16", "SASX", "SSAX",
        "UADD8", "UADD16", "USUB8", "USUB16", "UASX", "USAX", "t2SADD8",
        "t2SADD16", "t2SSUB8", "t2SSUB16", "t2SASX", "t2SSAX", "t2UADD8",
        "t2UADD16", "t2USUB8", "t2USUB16", "t2UASX", "t2USAX")>;
  // Flag setting.
  def : InstRW< [SwiftWriteP01TwoCycle],
       (instregex "SHADD8", "SHADD16", "SHSUB8", "SHSUB16", "SHASX", "SHSAX",
       "SXTAB", "SXTAB16", "SXTAH", "UHADD8", "UHADD16", "UHSUB8", "UHSUB16",
       "UHASX", "UHSAX", "UXTAB", "UXTAB16", "UXTAH", "t2SHADD8", "t2SHADD16",
       "t2SHSUB8", "t2SHSUB16", "t2SHASX", "t2SHSAX", "t2SXTAB", "t2SXTAB16",
       "t2SXTAH", "t2UHADD8", "t2UHADD16", "t2UHSUB8", "t2UHSUB16", "t2UHASX",
       "t2UHSAX", "t2UXTAB", "t2UXTAB16", "t2UXTAH")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 215-218
```tablegen
  // 4.2.11 Sum of Absolute Difference
  def : InstRW< [SwiftWriteP0P1FourCycle], (instregex "USAD8") >;
  def : InstRW<[SwiftWriteP0P1FourCycle, ReadALU, ReadALU, SchedReadAdvance<2>],
        (instregex "USADA8")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 220-226
```tablegen
  // 4.2.12 Integer Multiply (32-bit result)
  // Two sources.
  def : InstRW< [SwiftWriteP0FourCycle],
        (instregex "MUL", "SMMUL", "SMMULR", "SMULBB", "SMULBT",
        "SMULTB", "SMULTT", "SMULWB", "SMULWT", "SMUSD", "SMUSDX", "t2MUL",
        "t2SMMUL", "t2SMMULR", "t2SMULBB", "t2SMULBT", "t2SMULTB", "t2SMULTT",
        "t2SMULWB", "t2SMULWT", "t2SMUSD")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 228-231
```tablegen
  def SwiftWriteP0P01FiveCycleTwoUops :
      SchedWriteRes<[SwiftUnitP0, SwiftUnitP01]>  {
    let Latency = 5;
  }
```
- EN: Defines TableGen record `SwiftWriteP0P01FiveCycleTwoUops` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftWriteP0P01FiveCycleTwoUops`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 233-236
```tablegen
  def SwiftPredP0P01FourFiveCycle : SchedWriteVariant<[
    SchedVar<IsPredicatedPred, [ SwiftWriteP0P01FiveCycleTwoUops ]>,
    SchedVar<NoSchedPred,      [ SwiftWriteP0FourCycle ]>
  ]>;
```
- EN: Defines TableGen record `SwiftPredP0P01FourFiveCycle` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftPredP0P01FourFiveCycle`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 238-241
```tablegen
  def SwiftReadAdvanceFourCyclesPred : SchedReadVariant<[
     SchedVar<IsPredicatedPred, [SchedReadAdvance<4>]>,
     SchedVar<NoSchedPred,      [ReadALU]>
  ]>;
```
- EN: Defines TableGen record `SwiftReadAdvanceFourCyclesPred` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftReadAdvanceFourCyclesPred`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 243-248
```tablegen
  // Multiply accumulate, three sources
  def : InstRW< [SwiftPredP0P01FourFiveCycle, ReadALU, ReadALU,
                 SwiftReadAdvanceFourCyclesPred],
        (instregex "MLA", "MLS", "SMMLA", "SMMLAR", "SMMLS", "SMMLSR",
        "t2MLA", "t2MLS", "t2SMMLA", "t2SMMLAR", "t2SMMLS",
        "t2SMMLSR")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 250-259
```tablegen
  // 4.2.13 Integer Multiply (32-bit result, Q flag)
  def : InstRW< [SwiftWriteP0FourCycle],
        (instregex "SMUAD", "SMUADX", "t2SMUAD", "t2SMUADX")>;
  def : InstRW< [SwiftPredP0P01FourFiveCycle, ReadALU, ReadALU,
                 SwiftReadAdvanceFourCyclesPred],
        (instregex "SMLABB", "SMLABT", "SMLATB", "SMLATT", "SMLSD", "SMLSDX",
        "SMLAWB", "SMLAWT", "t2SMLABB", "t2SMLABT", "t2SMLATB", "t2SMLATT",
        "t2SMLSD", "t2SMLSDX", "t2SMLAWB", "t2SMLAWT")>;
  def : InstRW< [SwiftPredP0P01FourFiveCycle],
        (instregex "SMLAD", "SMLADX", "t2SMLAD", "t2SMLADX")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 261-277
```tablegen
  def SwiftP0P0P01FiveCycle : SchedWriteRes<[SwiftUnitP0, SwiftUnitP01]> {
    let Latency = 5;
    let NumMicroOps = 3;
    let ReleaseAtCycles = [2, 1];
  }
  def SwiftWrite1Cycle : SchedWriteRes<[]> {
    let Latency = 1;
    let NumMicroOps = 0;
  }
  def SwiftWrite5Cycle : SchedWriteRes<[]> {
    let Latency = 5;
    let NumMicroOps = 0;
  }
  def SwiftWrite6Cycle : SchedWriteRes<[]> {
    let Latency = 6;
    let NumMicroOps = 0;
  }
```
- EN: Defines TableGen record `SwiftP0P0P01FiveCycle` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftP0P0P01FiveCycle`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 279-281
```tablegen
  // 4.2.14 Integer Multiply, Long
  def : InstRW< [SwiftP0P0P01FiveCycle, SwiftWrite5Cycle],
        (instregex "SMULL$", "UMULL$", "t2SMULL$", "t2UMULL$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 283-287
```tablegen
  def Swift2P03P01FiveCycle : SchedWriteRes<[SwiftUnitP0, SwiftUnitP01]> {
    let Latency = 7;
    let NumMicroOps = 5;
    let ReleaseAtCycles = [2, 3];
  }
```
- EN: Defines TableGen record `Swift2P03P01FiveCycle` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Swift2P03P01FiveCycle`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 289-299
```tablegen
  // Aliasing sub-target specific WriteRes to generic ones
  def : SchedAlias<WriteMUL16, SwiftWriteP0FourCycle>;
  def : SchedAlias<WriteMUL32, SwiftWriteP0FourCycle>;
  def : SchedAlias<WriteMUL64Lo, SwiftP0P0P01FiveCycle>;
  def : SchedAlias<WriteMUL64Hi, SwiftWrite5Cycle>;
  def : SchedAlias<WriteMAC16, SwiftPredP0P01FourFiveCycle>;
  def : SchedAlias<WriteMAC32, SwiftPredP0P01FourFiveCycle>;
  def : SchedAlias<WriteMAC64Lo, SwiftWrite5Cycle>;
  def : SchedAlias<WriteMAC64Hi, Swift2P03P01FiveCycle>;
  def : ReadAdvance<ReadMUL, 0>;
  def : SchedAlias<ReadMAC, SwiftReadAdvanceFourCyclesPred>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 301-311
```tablegen
  // 4.2.15 Integer Multiply Accumulate, Long
  // 4.2.16 Integer Multiply Accumulate, Dual
  // 4.2.17 Integer Multiply Accumulate Accumulate, Long
  // We are being a bit inaccurate here.
  def : InstRW< [SwiftWrite5Cycle, Swift2P03P01FiveCycle, ReadALU, ReadALU,
                 SchedReadAdvance<4>, SchedReadAdvance<3>],
        (instregex "SMLAL", "UMLAL", "SMLALBT",
        "SMLALTB", "SMLALTT", "SMLALD", "SMLALDX", "SMLSLD", "SMLSLDX",
        "UMAAL", "t2SMLAL", "t2UMLAL", "t2SMLALBB", "t2SMLALBT",
        "t2SMLALTB", "t2SMLALTT", "t2SMLALD", "t2SMLALDX", "t2SMLSLD", "t2SMLSLDX",
        "t2UMAAL")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 313-321
```tablegen
  def SwiftDiv : SchedWriteRes<[SwiftUnitP0, SwiftUnitDiv]> {
    let NumMicroOps = 1;
    let Latency = 14;
    let ReleaseAtCycles = [1, 14];
  }
  // 4.2.18 Integer Divide
  def : WriteRes<WriteDIV, [SwiftUnitDiv]>; // Workaround.
  def : InstRW <[SwiftDiv],
        (instregex "SDIV", "UDIV", "t2SDIV", "t2UDIV")>;
```
- EN: Defines TableGen record `SwiftDiv` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftDiv`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 323-340
```tablegen
  // 4.2.19 Integer Load Single Element
  // 4.2.20 Integer Load Signextended
  def SwiftWriteP2P01ThreeCycle : SchedWriteRes<[SwiftUnitP2, SwiftUnitP01]> {
    let Latency = 3;
    let NumMicroOps = 2;
  }
  def SwiftWriteP2P01FourCycle : SchedWriteRes<[SwiftUnitP2, SwiftUnitP01]> {
    let Latency = 4;
    let NumMicroOps = 2;
  }
  def SwiftWriteP2P01P01FourCycle : SchedWriteRes<[SwiftUnitP2, SwiftUnitP01,
                                                   SwiftUnitP01]> {
    let Latency = 4;
    let NumMicroOps = 3;
  }
  def SwiftWriteP2P2ThreeCycle : SchedWriteRes<[SwiftUnitP2, SwiftUnitP2]> {
    let Latency = 3;
    let NumMicroOps = 2;
```
- EN: Defines TableGen record `SwiftWriteP2P01ThreeCycle` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftWriteP2P01ThreeCycle`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 341-358
```tablegen
  }
  def SwiftWriteP2P2P01ThreeCycle : SchedWriteRes<[SwiftUnitP2, SwiftUnitP2,
                                                   SwiftUnitP01]> {
    let Latency = 3;
    let NumMicroOps = 3;
  }
  def SwiftWrBackOne : SchedWriteRes<[]> {
    let Latency = 1;
    let NumMicroOps = 0;
  }
  def SwiftWriteLdFour : SchedWriteRes<[]> {
    let Latency = 4;
    let NumMicroOps = 0;
  }
   // Not accurate.
  def : InstRW<[SwiftWriteP2ThreeCycle],
        (instregex "LDR(i12|rs)$", "LDRB(i12|rs)$", "t2LDR(i8|i12|s|pci)",
        "t2LDR(H|B)(i8|i12|s|pci)", "LDREX", "tLDR[BH](r|i|spi|pci|pciASM)",
```
- EN: Defines TableGen record `SwiftWriteP2P2P01ThreeCycle` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftWriteP2P2P01ThreeCycle`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 359-371
```tablegen
        "tLDR(r|i|spi|pci|pciASM)")>;
  def : InstRW<[SwiftWriteP2ThreeCycle],
        (instregex "LDRH$",  "PICLDR$", "PICLDR(H|B)$", "LDRcp$")>;
  def : InstRW<[SwiftWriteP2P01FourCycle],
        (instregex "PICLDRS(H|B)$", "t2LDRS(H|B)(i|r|p|s)", "LDRS(H|B)$",
        "t2LDRpci_pic", "tLDRS(B|H)")>;
  def : InstRW<[SwiftWriteP2P01ThreeCycle,  SwiftWrBackOne],
        (instregex "LD(RB|R)(_|T_)(POST|PRE)_(IMM|REG)", "LDRH(_PRE|_POST)",
        "LDR(T|BT)_POST_(REG|IMM)", "LDRHT(i|r)",
        "t2LD(R|RB|RH)_(PRE|POST)", "t2LD(R|RB|RH)T")>;
  def : InstRW<[SwiftWriteP2P01P01FourCycle, SwiftWrBackOne],
        (instregex "LDR(SH|SB)(_POST|_PRE)", "t2LDR(SH|SB)(_POST|_PRE)",
        "LDRS(B|H)T(i|r)", "t2LDRS(B|H)T(i|r)?")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 373-378
```tablegen
  // 4.2.21 Integer Dual Load
  // Not accurate.
  def : InstRW<[SwiftWriteP2P2ThreeCycle, SwiftWriteLdFour],
        (instregex "t2LDRDi8", "LDRD$")>;
  def : InstRW<[SwiftWriteP2P2P01ThreeCycle, SwiftWriteLdFour, SwiftWrBackOne],
        (instregex "LDRD_(POST|PRE)", "t2LDRD_(POST|PRE)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 380-397
```tablegen
  // 4.2.22 Integer Load, Multiple
  // NumReg = 1 .. 16
  foreach Lat = 3-25 in {
    def SwiftWriteLM#Lat#Cy : SchedWriteRes<[SwiftUnitP2]> {
      let Latency = Lat;
    }
    def SwiftWriteLM#Lat#CyNo : SchedWriteRes<[]> {
      let Latency = Lat;
      let NumMicroOps = 0;
    }
  }
  // Predicate.
  foreach NumAddr = 1-16 in {
    def SwiftLMAddr#NumAddr#Pred : SchedPredicate<"TII->getNumLDMAddresses(*MI) == "#NumAddr>;
  }
  def SwiftWriteLDMAddrNoWB : SchedWriteRes<[SwiftUnitP01]> { let Latency = 0; }
  def SwiftWriteLDMAddrWB : SchedWriteRes<[SwiftUnitP01, SwiftUnitP01]>;
  def SwiftWriteLM : SchedWriteVariant<[
```
- EN: Defines TableGen record `SwiftWriteLM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftWriteLM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 398-415
```tablegen
    SchedVar<SwiftLMAddr2Pred, [SwiftWriteLM3Cy, SwiftWriteLM4Cy]>,
    SchedVar<SwiftLMAddr3Pred, [SwiftWriteLM3Cy, SwiftWriteLM4Cy,
                                SwiftWriteLM5Cy]>,
    SchedVar<SwiftLMAddr4Pred, [SwiftWriteLM3Cy, SwiftWriteLM4Cy,
                                SwiftWriteLM5Cy, SwiftWriteLM6Cy]>,
    SchedVar<SwiftLMAddr5Pred, [SwiftWriteLM3Cy, SwiftWriteLM4Cy,
                                SwiftWriteLM5Cy, SwiftWriteLM6Cy,
                                SwiftWriteLM7Cy]>,
    SchedVar<SwiftLMAddr6Pred, [SwiftWriteLM3Cy, SwiftWriteLM4Cy,
                                SwiftWriteLM5Cy, SwiftWriteLM6Cy,
                                SwiftWriteLM7Cy, SwiftWriteLM8Cy]>,
    SchedVar<SwiftLMAddr7Pred, [SwiftWriteLM3Cy, SwiftWriteLM4Cy,
                                SwiftWriteLM5Cy, SwiftWriteLM6Cy,
                                SwiftWriteLM7Cy, SwiftWriteLM8Cy,
                                SwiftWriteLM9Cy]>,
    SchedVar<SwiftLMAddr8Pred, [SwiftWriteLM3Cy, SwiftWriteLM4Cy,
                                SwiftWriteLM5Cy, SwiftWriteLM6Cy,
                                SwiftWriteLM7Cy, SwiftWriteLM8Cy,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 416-433
```tablegen
                                SwiftWriteLM9Cy, SwiftWriteLM10Cy]>,
    SchedVar<SwiftLMAddr9Pred, [SwiftWriteLM3Cy, SwiftWriteLM4Cy,
                                SwiftWriteLM5Cy, SwiftWriteLM6Cy,
                                SwiftWriteLM7Cy, SwiftWriteLM8Cy,
                                SwiftWriteLM9Cy, SwiftWriteLM10Cy,
                                SwiftWriteLM11Cy]>,
    SchedVar<SwiftLMAddr10Pred,[SwiftWriteLM3Cy, SwiftWriteLM4Cy,
                                SwiftWriteLM5Cy, SwiftWriteLM6Cy,
                                SwiftWriteLM7Cy, SwiftWriteLM8Cy,
                                SwiftWriteLM9Cy, SwiftWriteLM10Cy,
                                SwiftWriteLM11Cy, SwiftWriteLM12Cy]>,
    SchedVar<SwiftLMAddr11Pred,[SwiftWriteLM3Cy, SwiftWriteLM4Cy,
                                SwiftWriteLM5Cy, SwiftWriteLM6Cy,
                                SwiftWriteLM7Cy, SwiftWriteLM8Cy,
                                SwiftWriteLM9Cy, SwiftWriteLM10Cy,
                                SwiftWriteLM11Cy, SwiftWriteLM12Cy,
                                SwiftWriteLM13Cy]>,
    SchedVar<SwiftLMAddr12Pred,[SwiftWriteLM3Cy, SwiftWriteLM4Cy,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 434-451
```tablegen
                                SwiftWriteLM5Cy, SwiftWriteLM6Cy,
                                SwiftWriteLM7Cy, SwiftWriteLM8Cy,
                                SwiftWriteLM9Cy, SwiftWriteLM10Cy,
                                SwiftWriteLM11Cy, SwiftWriteLM12Cy,
                                SwiftWriteLM13Cy, SwiftWriteLM14Cy]>,
    SchedVar<SwiftLMAddr13Pred,[SwiftWriteLM3Cy, SwiftWriteLM4Cy,
                                SwiftWriteLM5Cy, SwiftWriteLM6Cy,
                                SwiftWriteLM7Cy, SwiftWriteLM8Cy,
                                SwiftWriteLM9Cy, SwiftWriteLM10Cy,
                                SwiftWriteLM11Cy, SwiftWriteLM12Cy,
                                SwiftWriteLM13Cy, SwiftWriteLM14Cy,
                                SwiftWriteLM15Cy]>,
    SchedVar<SwiftLMAddr14Pred,[SwiftWriteLM3Cy, SwiftWriteLM4Cy,
                                SwiftWriteLM5Cy, SwiftWriteLM6Cy,
                                SwiftWriteLM7Cy, SwiftWriteLM8Cy,
                                SwiftWriteLM9Cy, SwiftWriteLM10Cy,
                                SwiftWriteLM11Cy, SwiftWriteLM12Cy,
                                SwiftWriteLM13Cy, SwiftWriteLM14Cy,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 452-469
```tablegen
                                SwiftWriteLM15Cy, SwiftWriteLM16Cy]>,
    SchedVar<SwiftLMAddr15Pred,[SwiftWriteLM3Cy, SwiftWriteLM4Cy,
                                SwiftWriteLM5Cy, SwiftWriteLM6Cy,
                                SwiftWriteLM7Cy, SwiftWriteLM8Cy,
                                SwiftWriteLM9Cy, SwiftWriteLM10Cy,
                                SwiftWriteLM11Cy, SwiftWriteLM12Cy,
                                SwiftWriteLM13Cy, SwiftWriteLM14Cy,
                                SwiftWriteLM15Cy, SwiftWriteLM16Cy,
                                SwiftWriteLM17Cy]>,
    SchedVar<SwiftLMAddr16Pred,[SwiftWriteLM3Cy, SwiftWriteLM4Cy,
                                SwiftWriteLM5Cy, SwiftWriteLM6Cy,
                                SwiftWriteLM7Cy, SwiftWriteLM8Cy,
                                SwiftWriteLM9Cy, SwiftWriteLM10Cy,
                                SwiftWriteLM11Cy, SwiftWriteLM12Cy,
                                SwiftWriteLM13Cy, SwiftWriteLM14Cy,
                                SwiftWriteLM15Cy, SwiftWriteLM16Cy,
                                SwiftWriteLM17Cy, SwiftWriteLM18Cy]>,
    // Unknow number of registers, just use resources for two registers.
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 470-477
```tablegen
    SchedVar<NoSchedPred,      [SwiftWriteLM3Cy, SwiftWriteLM4Cy,
                                SwiftWriteLM5CyNo, SwiftWriteLM6CyNo,
                                SwiftWriteLM7CyNo, SwiftWriteLM8CyNo,
                                SwiftWriteLM9CyNo, SwiftWriteLM10CyNo,
                                SwiftWriteLM11CyNo, SwiftWriteLM12CyNo,
                                SwiftWriteLM13CyNo, SwiftWriteLM14CyNo,
                                SwiftWriteLM15CyNo, SwiftWriteLM16CyNo,
                                SwiftWriteLM17CyNo, SwiftWriteLM18CyNo]>
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 479-479
```tablegen
  ]> { let Variadic=1; }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 481-492
```tablegen
  def : InstRW<[SwiftWriteLM, SwiftWriteLDMAddrNoWB],
        (instregex "LDM(IA|DA|DB|IB)$", "t2LDM(IA|DA|DB|IB)$",
        "(t|sys)LDM(IA|DA|DB|IB)$")>;
  def : InstRW<[SwiftWriteLDMAddrWB, SwiftWriteLM],
        (instregex /*"t2LDMIA_RET", "tLDMIA_RET", "LDMIA_RET",*/
        "LDM(IA|DA|DB|IB)_UPD", "(t2|sys|t)LDM(IA|DA|DB|IB)_UPD")>;
  def : InstRW<[SwiftWriteLDMAddrWB, SwiftWriteLM, SwiftWriteP1TwoCycle],
        (instregex "LDMIA_RET", "(t|t2)LDMIA_RET", "tPOP")>;
  // 4.2.23 Integer Store, Single Element
  def : InstRW<[SwiftWriteP2],
        (instregex "PICSTR", "STR(i12|rs)", "STRB(i12|rs)", "STRH$", "STREX",
        "t2STR(i12|i8|s)$", "t2STR[BH](i12|i8|s)$", "tSTR[BH](i|r)", "tSTR(i|r)", "tSTRspi")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 494-498
```tablegen
  def : InstRW<[SwiftWriteP01OneCycle, SwiftWriteP2],
        (instregex "STR(B_|_|BT_|T_)(PRE_IMM|PRE_REG|POST_REG|POST_IMM)",
        "STR(i|r)_preidx", "STRB(i|r)_preidx", "STRH_preidx", "STR(H_|HT_)(PRE|POST)",
        "STR(BT|HT|T)", "t2STR_(PRE|POST)", "t2STR[BH]_(PRE|POST)",
        "t2STR_preidx", "t2STR[BH]_preidx", "t2ST(RB|RH|R)T")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 500-505
```tablegen
  // 4.2.24 Integer Store, Dual
  def : InstRW<[SwiftWriteP2, SwiftWriteP2, SwiftWriteP01OneCycle],
        (instregex "STRD$", "t2STRDi8")>;
  def : InstRW<[SwiftWriteP01OneCycle, SwiftWriteP2, SwiftWriteP2,
                SwiftWriteP01OneCycle],
        (instregex "(t2|t)STRD_(POST|PRE)", "STRD_(POST|PRE)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 507-524
```tablegen
  // 4.2.25 Integer Store, Multiple
  def SwiftWriteStIncAddr : SchedWriteRes<[SwiftUnitP2, SwiftUnitP01]> {
    let Latency = 0;
    let NumMicroOps = 2;
  }
  foreach NumAddr = 1-16 in {
     def SwiftWriteSTM#NumAddr : WriteSequence<[SwiftWriteStIncAddr], NumAddr>;
  }
  def SwiftWriteSTM : SchedWriteVariant<[
    SchedVar<SwiftLMAddr2Pred, [SwiftWriteSTM2]>,
    SchedVar<SwiftLMAddr3Pred, [SwiftWriteSTM3]>,
    SchedVar<SwiftLMAddr4Pred, [SwiftWriteSTM4]>,
    SchedVar<SwiftLMAddr5Pred, [SwiftWriteSTM5]>,
    SchedVar<SwiftLMAddr6Pred, [SwiftWriteSTM6]>,
    SchedVar<SwiftLMAddr7Pred, [SwiftWriteSTM7]>,
    SchedVar<SwiftLMAddr8Pred, [SwiftWriteSTM8]>,
    SchedVar<SwiftLMAddr9Pred, [SwiftWriteSTM9]>,
    SchedVar<SwiftLMAddr10Pred,[SwiftWriteSTM10]>,
```
- EN: Defines TableGen record `SwiftWriteStIncAddr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftWriteStIncAddr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 525-538
```tablegen
    SchedVar<SwiftLMAddr11Pred,[SwiftWriteSTM11]>,
    SchedVar<SwiftLMAddr12Pred,[SwiftWriteSTM12]>,
    SchedVar<SwiftLMAddr13Pred,[SwiftWriteSTM13]>,
    SchedVar<SwiftLMAddr14Pred,[SwiftWriteSTM14]>,
    SchedVar<SwiftLMAddr15Pred,[SwiftWriteSTM15]>,
    SchedVar<SwiftLMAddr16Pred,[SwiftWriteSTM16]>,
    // Unknow number of registers, just use resources for two registers.
    SchedVar<NoSchedPred,      [SwiftWriteSTM2]>
  ]>;
  def : InstRW<[SwiftWriteSTM],
        (instregex "STM(IB|IA|DB|DA)$", "(t2|sys|t)STM(IB|IA|DB|DA)$")>;
  def : InstRW<[SwiftWriteP01OneCycle, SwiftWriteSTM],
        (instregex "STM(IB|IA|DB|DA)_UPD", "(t2|sys|t)STM(IB|IA|DB|DA)_UPD",
        "tPUSH")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 540-545
```tablegen
  // LDRLIT pseudo instructions, they expand to LDR + PICADD
  def : InstRW<[SwiftWriteP2ThreeCycle, WriteALU],
        (instregex "t?LDRLIT_ga_abs", "t?LDRLIT_ga_pcrel")>;
  // LDRLIT_ga_pcrel_ldr expands to LDR + PICLDR
  def : InstRW<[SwiftWriteP2ThreeCycle, SwiftWriteP2ThreeCycle],
        (instregex "LDRLIT_ga_pcrel_ldr")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 547-550
```tablegen
  // 4.2.26 Branch
  def : WriteRes<WriteBr, [SwiftUnitP1]> { let Latency = 0; }
  def : WriteRes<WriteBrL, [SwiftUnitP1]> { let Latency = 2; }
  def : WriteRes<WriteBrTbl, [SwiftUnitP1, SwiftUnitP2]> { let Latency = 0; }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 552-554
```tablegen
  // 4.2.27 Not issued
  def : WriteRes<WriteNoop, []> { let Latency = 0; let NumMicroOps = 0; }
  def : InstRW<[WriteNoop], (instregex "t2IT", "IT")>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 556-562
```tablegen
  // 4.2.28 Advanced SIMD, Integer, 2 cycle
  def : InstRW<[SwiftWriteP0TwoCycle],
        (instregex "VADDv", "VSUBv", "VNEG(s|f|v)", "VADDL", "VSUBL",
                   "VADDW", "VSUBW", "VHADD", "VHSUB", "VRHADD", "VPADDi",
                   "VPADDL", "VAND", "VBIC", "VEOR", "VORN", "VORR", "VTST",
                   "VSHL", "VSHR(s|u)", "VSHLL", "VQSHL(s|u)", "VBIF", "VBIT",
                   "VBSL", "VBSP", "VSLI", "VSRI", "VCLS", "VCLZ", "VCNT")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 564-565
```tablegen
  def : InstRW<[SwiftWriteP1TwoCycle],
        (instregex "VEXT", "VREV16", "VREV32", "VREV64")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 567-575
```tablegen
  // 4.2.29 Advanced SIMD, Integer, 4 cycle
  // 4.2.30 Advanced SIMD, Integer with Accumulate
  def : InstRW<[SwiftWriteP0FourCycle],
        (instregex "VABA", "VABAL", "VPADAL", "VRSRA", "VSRA", "VACGE", "VACGT",
        "VCEQ", "VCGE", "VCGT", "VCLE", "VCLT", "VRSHL",
        "VQRSHL", "VRSHR(u|s)", "VABS(f|v)", "VQABS", "VQNEG", "VQADD",
        "VQSUB")>;
  def : InstRW<[SwiftWriteP1FourCycle],
        (instregex "VRECPE", "VRSQRTE")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 577-582
```tablegen
  // 4.2.31 Advanced SIMD, Add and Shift with Narrow
  def : InstRW<[SwiftWriteP0P1FourCycle],
        (instregex "VADDHN", "VSUBHN", "VSHRN")>;
  def : InstRW<[SwiftWriteP0P1SixCycle],
        (instregex "VRADDHN", "VRSUBHN", "VRSHRN", "VQSHRN", "VQSHRUN",
                   "VQRSHRN", "VQRSHRUN")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 584-595
```tablegen
  // 4.2.32 Advanced SIMD, Vector Table Lookup
  foreach Num = 1-4 in {
    def SwiftWrite#Num#xP1TwoCycle : WriteSequence<[SwiftWriteP1TwoCycle], Num>;
  }
  def : InstRW<[SwiftWrite1xP1TwoCycle],
        (instregex "VTB(L|X)1")>;
  def : InstRW<[SwiftWrite2xP1TwoCycle],
        (instregex "VTB(L|X)2")>;
  def : InstRW<[SwiftWrite3xP1TwoCycle],
        (instregex "VTB(L|X)3")>;
  def : InstRW<[SwiftWrite4xP1TwoCycle],
        (instregex "VTB(L|X)4")>;
```
- EN: Defines TableGen record `SwiftWrite` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftWrite`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 597-600
```tablegen
  // 4.2.33 Advanced SIMD, Transpose
  def : InstRW<[SwiftWriteP1FourCycle, SwiftWriteP1FourCycle,
                SwiftWriteP1TwoCycle/*RsrcOnly*/, SchedReadAdvance<2>],
        (instregex "VSWP", "VTRN", "VUZP", "VZIP")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 602-610
```tablegen
  // 4.2.34 Advanced SIMD and VFP, Floating Point
  def : InstRW<[SwiftWriteP0TwoCycle], (instregex "VABS(S|D)$", "VNEG(S|D)$")>;
  def : InstRW<[SwiftWriteP0FourCycle],
        (instregex "VCMP(D|S|ZD|ZS)$", "VCMPE(D|S|ZD|ZS)")>;
  def : InstRW<[SwiftWriteP0FourCycle],
        (instregex "VADD(S|f)", "VSUB(S|f)", "VABD", "VPADDf", "VMAX", "VMIN", "VPMAX",
                   "VPMIN")>;
  def : InstRW<[SwiftWriteP0SixCycle], (instregex "VADDD$", "VSUBD$")>;
  def : InstRW<[SwiftWriteP1EightCycle], (instregex "VRECPS", "VRSQRTS")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 612-620
```tablegen
  // 4.2.35 Advanced SIMD and VFP, Multiply
  def : InstRW<[SwiftWriteP1FourCycle],
        (instregex "VMUL(S|v|p|f|s)", "VNMULS", "VQDMULH", "VQRDMULH",
                   "VMULL", "VQDMULL")>;
  def : InstRW<[SwiftWriteP1FourCycle],
        (instregex "VMLA", "VMLS", "VNMLA", "VNMLS", "VFMA(S|D)", "VFMS(S|D)",
        "VFNMA", "VFNMS", "VMLAL", "VMLSL","VQDMLAL", "VQDMLSL")>;
  def : InstRW<[SwiftWriteP1EightCycle], (instregex "VFMAfd", "VFMSfd")>;
  def : InstRW<[SwiftWriteP1TwelveCyc], (instregex "VFMAfq", "VFMSfq")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 622-623
```tablegen
  // 4.2.36 Advanced SIMD and VFP, Convert
  def : InstRW<[SwiftWriteP1FourCycle], (instregex "VCVT", "V(S|U)IT", "VTO(S|U)")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 625-642
```tablegen
  // 4.2.37 Advanced SIMD and VFP, Move
  def : InstRW<[SwiftWriteP0TwoCycle],
        (instregex "VMOVv", "VMOV(S|D)$", "VMOV(S|D)cc",
                   "VMVNv", "VMVN(d|q)",
                   "FCONST(D|S)")>;
  def : InstRW<[SwiftWriteP1TwoCycle], (instregex "VMOVN", "VMOVL")>;
  def : InstRW<[WriteSequence<[SwiftWriteP0FourCycle, SwiftWriteP1TwoCycle]>],
        (instregex "VQMOVN")>;
  def : InstRW<[SwiftWriteP1TwoCycle], (instregex "VDUPLN")>;
  def : InstRW<[WriteSequence<[SwiftWriteP2FourCycle, SwiftWriteP1TwoCycle]>],
        (instregex "VDUP(8|16|32)")>;
  def : InstRW<[SwiftWriteP2ThreeCycle], (instregex "VMOVRS$")>;
  def : InstRW<[WriteSequence<[SwiftWriteP2FourCycle, SwiftWriteP0TwoCycle]>],
        (instregex "VMOVSR$", "VSETLN")>;
  def : InstRW<[SwiftWriteP2ThreeCycle, SwiftWriteP2FourCycle],
        (instregex "VMOVRR(D|S)$")>;
  def : InstRW<[SwiftWriteP2FourCycle], (instregex "VMOVDRR$")>;
  def : InstRW<[WriteSequence<[SwiftWriteP2FourCycle, SwiftWriteP1TwoCycle]>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 643-650
```tablegen
                WriteSequence<[SwiftWrite1Cycle, SwiftWriteP2FourCycle,
                               SwiftWriteP1TwoCycle]>],
                (instregex "VMOVSRR$")>;
  def : InstRW<[WriteSequence<[SwiftWriteP1TwoCycle, SwiftWriteP2ThreeCycle]>],
        (instregex "VGETLN(u|i)")>;
  def : InstRW<[WriteSequence<[SwiftWriteP1TwoCycle, SwiftWriteP2ThreeCycle,
                               SwiftWriteP01OneCycle]>],
        (instregex "VGETLNs")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 652-669
```tablegen
  // 4.2.38 Advanced SIMD and VFP, Move FPSCR
  // Serializing instructions.
  def SwiftWaitP0For15Cy : SchedWriteRes<[SwiftUnitP0]> {
    let Latency = 15;
    let ReleaseAtCycles = [15];
  }
  def SwiftWaitP1For15Cy : SchedWriteRes<[SwiftUnitP1]> {
    let Latency = 15;
    let ReleaseAtCycles = [15];
  }
  def SwiftWaitP2For15Cy : SchedWriteRes<[SwiftUnitP2]> {
    let Latency = 15;
    let ReleaseAtCycles = [15];
  }
  def : InstRW<[SwiftWaitP0For15Cy, SwiftWaitP1For15Cy, SwiftWaitP2For15Cy],
        (instregex "VMRS")>;
  def : InstRW<[SwiftWaitP0For15Cy, SwiftWaitP1For15Cy, SwiftWaitP2For15Cy],
        (instregex "VMSR")>;
```
- EN: Defines TableGen record `SwiftWaitP0For15Cy` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftWaitP0For15Cy`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 670-671
```tablegen
  // Not serializing.
  def : InstRW<[SwiftWriteP0TwoCycle], (instregex "FMSTAT")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 673-674
```tablegen
  // 4.2.39 Advanced SIMD and VFP, Load Single Element
  def : InstRW<[SwiftWriteLM4Cy], (instregex "VLDRD$", "VLDRS$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 676-677
```tablegen
  // 4.2.40 Advanced SIMD and VFP, Store Single Element
  def : InstRW<[SwiftWriteLM4Cy], (instregex "VSTRD$", "VSTRS$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 679-680
```tablegen
  // 4.2.41 Advanced SIMD and VFP, Load Multiple
  // 4.2.42 Advanced SIMD and VFP, Store Multiple
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 682-689
```tablegen
  // Resource requirement for permuting, just reserves the resources.
  foreach Num = 1-28 in {
    def SwiftVLDMPerm#Num : SchedWriteRes<[SwiftUnitP1]> {
      let Latency = 0;
      let NumMicroOps = Num;
      let ReleaseAtCycles = [Num];
    }
  }
```
- EN: Defines TableGen record `SwiftVLDMPerm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftVLDMPerm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 691-692
```tablegen
  // Pre RA pseudos - load/store to a Q register as a D register pair.
  def : InstRW<[SwiftWriteLM4Cy], (instregex "VLDMQIA$", "VSTMQIA$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 694-711
```tablegen
  // Post RA not modelled accurately. We assume that register use of width 64
  // bit maps to a D register, 128 maps to a Q register. Not all different kinds
  // are accurately represented.
  def SwiftWriteVLDM : SchedWriteVariant<[
    // Load of one S register.
    SchedVar<SwiftLMAddr1Pred, [SwiftWriteLM4Cy]>,
    // Load of one D register.
    SchedVar<SwiftLMAddr2Pred, [SwiftWriteLM4Cy, SwiftWriteLM4CyNo]>,
    // Load of 3 S register.
    SchedVar<SwiftLMAddr3Pred, [SwiftWriteLM9Cy, SwiftWriteLM10Cy,
                                SwiftWriteLM13CyNo, SwiftWriteP01OneCycle,
                                SwiftVLDMPerm3]>,
    // Load of a Q register (not necessarily true). We should not be mapping to
    // 4 S registers, either.
    SchedVar<SwiftLMAddr4Pred, [SwiftWriteLM4Cy, SwiftWriteLM4CyNo,
                                SwiftWriteLM4CyNo, SwiftWriteLM4CyNo]>,
    // Load of 5 S registers.
    SchedVar<SwiftLMAddr5Pred, [SwiftWriteLM9Cy, SwiftWriteLM10Cy,
```
- EN: Defines TableGen record `SwiftWriteVLDM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftWriteVLDM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 712-729
```tablegen
                                SwiftWriteLM13CyNo, SwiftWriteLM14CyNo,
                                SwiftWriteLM17CyNo,  SwiftWriteP01OneCycle,
                                SwiftVLDMPerm5]>,
    // Load of 3 D registers. (Must also be able to handle s register list -
    // though, not accurate)
    SchedVar<SwiftLMAddr6Pred, [SwiftWriteLM7Cy, SwiftWriteLM8Cy,
                                SwiftWriteLM10Cy, SwiftWriteLM14CyNo,
                                SwiftWriteLM14CyNo, SwiftWriteLM14CyNo,
                                SwiftWriteP01OneCycle, SwiftVLDMPerm5]>,
    // Load of 7 S registers.
    SchedVar<SwiftLMAddr7Pred, [SwiftWriteLM9Cy, SwiftWriteLM10Cy,
                                SwiftWriteLM13Cy, SwiftWriteLM14CyNo,
                                SwiftWriteLM17CyNo, SwiftWriteLM18CyNo,
                                SwiftWriteLM21CyNo, SwiftWriteP01OneCycle,
                                SwiftVLDMPerm7]>,
    // Load of two Q registers.
    SchedVar<SwiftLMAddr8Pred, [SwiftWriteLM7Cy, SwiftWriteLM8Cy,
                                SwiftWriteLM13Cy, SwiftWriteLM13CyNo,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 730-747
```tablegen
                                SwiftWriteLM13CyNo, SwiftWriteLM13CyNo,
                                SwiftWriteLM13CyNo, SwiftWriteLM13CyNo,
                                SwiftWriteP01OneCycle,  SwiftVLDMPerm2]>,
    // Load of 9 S registers.
    SchedVar<SwiftLMAddr9Pred, [SwiftWriteLM9Cy, SwiftWriteLM10Cy,
                                SwiftWriteLM13Cy, SwiftWriteLM14CyNo,
                                SwiftWriteLM17CyNo, SwiftWriteLM18CyNo,
                                SwiftWriteLM21CyNo, SwiftWriteLM22CyNo,
                                SwiftWriteLM25CyNo, SwiftWriteP01OneCycle,
                                SwiftVLDMPerm9]>,
    // Load of 5 D registers.
    SchedVar<SwiftLMAddr10Pred,[SwiftWriteLM7Cy, SwiftWriteLM8Cy,
                                SwiftWriteLM10Cy, SwiftWriteLM14Cy,
                                SwiftWriteLM14CyNo, SwiftWriteLM14CyNo,
                                SwiftWriteLM14CyNo, SwiftWriteLM14CyNo,
                                SwiftWriteLM14CyNo,  SwiftWriteLM14CyNo,
                                SwiftWriteP01OneCycle, SwiftVLDMPerm5]>,
    // Inaccurate: reuse description from 9 S registers.
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 748-765
```tablegen
    SchedVar<SwiftLMAddr11Pred,[SwiftWriteLM9Cy, SwiftWriteLM10Cy,
                                SwiftWriteLM13Cy, SwiftWriteLM14CyNo,
                                SwiftWriteLM17CyNo, SwiftWriteLM18CyNo,
                                SwiftWriteLM21CyNo, SwiftWriteLM22CyNo,
                                SwiftWriteLM21CyNo, SwiftWriteLM22CyNo,
                                SwiftWriteLM25CyNo, SwiftWriteP01OneCycle,
                                SwiftVLDMPerm9]>,
    // Load of three Q registers.
    SchedVar<SwiftLMAddr12Pred,[SwiftWriteLM7Cy, SwiftWriteLM8Cy,
                                SwiftWriteLM11Cy, SwiftWriteLM11Cy,
                                SwiftWriteLM11CyNo, SwiftWriteLM11CyNo,
                                SwiftWriteLM11CyNo, SwiftWriteLM11CyNo,
                                SwiftWriteLM11CyNo, SwiftWriteLM11CyNo,
                                SwiftWriteLM11CyNo, SwiftWriteLM11CyNo,
                                SwiftWriteP01OneCycle, SwiftVLDMPerm3]>,
    // Inaccurate: reuse description from 9 S registers.
    SchedVar<SwiftLMAddr13Pred, [SwiftWriteLM9Cy, SwiftWriteLM10Cy,
                                SwiftWriteLM13Cy, SwiftWriteLM14CyNo,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 766-783
```tablegen
                                SwiftWriteLM17CyNo, SwiftWriteLM18CyNo,
                                SwiftWriteLM21CyNo, SwiftWriteLM22CyNo,
                                SwiftWriteLM21CyNo, SwiftWriteLM22CyNo,
                                SwiftWriteLM21CyNo, SwiftWriteLM22CyNo,
                                SwiftWriteLM25CyNo, SwiftWriteP01OneCycle,
                                SwiftVLDMPerm9]>,
    // Load of 7 D registers inaccurate.
    SchedVar<SwiftLMAddr14Pred,[SwiftWriteLM7Cy, SwiftWriteLM8Cy,
                                SwiftWriteLM10Cy, SwiftWriteLM14Cy,
                                SwiftWriteLM14Cy, SwiftWriteLM14CyNo,
                                SwiftWriteLM14CyNo, SwiftWriteLM14CyNo,
                                SwiftWriteLM14CyNo,  SwiftWriteLM14CyNo,
                                SwiftWriteLM14CyNo,  SwiftWriteLM14CyNo,
                                SwiftWriteP01OneCycle, SwiftVLDMPerm7]>,
    SchedVar<SwiftLMAddr15Pred,[SwiftWriteLM9Cy, SwiftWriteLM10Cy,
                                SwiftWriteLM13Cy, SwiftWriteLM14Cy,
                                SwiftWriteLM17Cy, SwiftWriteLM18CyNo,
                                SwiftWriteLM21CyNo, SwiftWriteLM22CyNo,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 784-801
```tablegen
                                SwiftWriteLM21CyNo, SwiftWriteLM22CyNo,
                                SwiftWriteLM21CyNo, SwiftWriteLM22CyNo,
                                SwiftWriteLM21CyNo, SwiftWriteLM22CyNo,
                                SwiftWriteLM25CyNo, SwiftWriteP01OneCycle,
                                SwiftVLDMPerm9]>,
    // Load of 4 Q registers.
    SchedVar<SwiftLMAddr16Pred,[SwiftWriteLM7Cy, SwiftWriteLM10Cy,
                                SwiftWriteLM11Cy, SwiftWriteLM14Cy,
                                SwiftWriteLM15Cy, SwiftWriteLM18CyNo,
                                SwiftWriteLM19CyNo, SwiftWriteLM22CyNo,
                                SwiftWriteLM19CyNo, SwiftWriteLM22CyNo,
                                SwiftWriteLM19CyNo, SwiftWriteLM22CyNo,
                                SwiftWriteLM19CyNo, SwiftWriteLM22CyNo,
                                SwiftWriteLM19CyNo, SwiftWriteLM22CyNo,
                                SwiftWriteP01OneCycle, SwiftVLDMPerm4]>,
    // Unknow number of registers, just use resources for two registers.
    SchedVar<NoSchedPred,      [SwiftWriteLM7Cy, SwiftWriteLM8Cy,
                                SwiftWriteLM13Cy, SwiftWriteLM13CyNo,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 802-817
```tablegen
                                SwiftWriteLM13CyNo, SwiftWriteLM13CyNo,
                                SwiftWriteLM13CyNo, SwiftWriteLM13CyNo,
                                SwiftWriteLM13CyNo, SwiftWriteLM13CyNo,
                                SwiftWriteLM13CyNo, SwiftWriteLM13CyNo,
                                SwiftWriteLM13CyNo, SwiftWriteLM13CyNo,
                                SwiftWriteLM13CyNo, SwiftWriteLM13CyNo,
                                SwiftWriteLM13CyNo, SwiftWriteLM13CyNo,
                                SwiftWriteLM13CyNo, SwiftWriteLM13CyNo,
                                SwiftWriteLM13CyNo, SwiftWriteLM13CyNo,
                                SwiftWriteLM13CyNo, SwiftWriteLM13CyNo,
                                SwiftWriteLM13CyNo, SwiftWriteLM13CyNo,
                                SwiftWriteLM13CyNo, SwiftWriteLM13CyNo,
                                SwiftWriteLM13CyNo, SwiftWriteLM13CyNo,
                                SwiftWriteLM13CyNo, SwiftWriteLM13CyNo,
                                SwiftWriteP01OneCycle,  SwiftVLDMPerm2]>
  ]> { let Variadic = 1; }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 819-819
```tablegen
  def : InstRW<[SwiftWriteVLDM], (instregex "VLDM[SD](IA|DB)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 821-822
```tablegen
  def : InstRW<[SwiftWriteP01OneCycle2x, SwiftWriteVLDM],
        (instregex "VLDM[SD](IA|DB)_UPD$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 824-841
```tablegen
  def SwiftWriteVSTM : SchedWriteVariant<[
    // One S register.
    SchedVar<SwiftLMAddr1Pred, [SwiftWriteSTM1]>,
    // One D register.
    SchedVar<SwiftLMAddr2Pred, [SwiftWriteSTM1]>,
    // Three S registers.
    SchedVar<SwiftLMAddr3Pred, [SwiftWriteSTM4]>,
    // Assume one Q register.
    SchedVar<SwiftLMAddr4Pred, [SwiftWriteSTM1]>,
    SchedVar<SwiftLMAddr5Pred, [SwiftWriteSTM6]>,
    // Assume three D registers.
    SchedVar<SwiftLMAddr6Pred, [SwiftWriteSTM4]>,
    SchedVar<SwiftLMAddr7Pred, [SwiftWriteSTM8]>,
    // Assume two Q registers.
    SchedVar<SwiftLMAddr8Pred, [SwiftWriteSTM3]>,
    SchedVar<SwiftLMAddr9Pred, [SwiftWriteSTM10]>,
    // Assume 5 D registers.
    SchedVar<SwiftLMAddr10Pred, [SwiftWriteSTM6]>,
```
- EN: Defines TableGen record `SwiftWriteVSTM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftWriteVSTM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 842-853
```tablegen
    SchedVar<SwiftLMAddr11Pred, [SwiftWriteSTM12]>,
    // Assume three Q registers.
    SchedVar<SwiftLMAddr12Pred, [SwiftWriteSTM4]>,
    SchedVar<SwiftLMAddr13Pred, [SwiftWriteSTM14]>,
    // Assume 7 D registers.
    SchedVar<SwiftLMAddr14Pred, [SwiftWriteSTM8]>,
    SchedVar<SwiftLMAddr15Pred, [SwiftWriteSTM16]>,
    // Assume four Q registers.
    SchedVar<SwiftLMAddr16Pred, [SwiftWriteSTM5]>,
    // Asumme two Q registers.
    SchedVar<NoSchedPred, [SwiftWriteSTM3]>
  ]> { let Variadic = 1; }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 855-855
```tablegen
  def : InstRW<[SwiftWriteVSTM], (instregex "VSTM[SD](IA|DB)$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 857-858
```tablegen
  def : InstRW<[SwiftWriteP01OneCycle2x, SwiftWriteVSTM],
        (instregex "VSTM[SD](IA|DB)_UPD")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 860-877
```tablegen
  // 4.2.43 Advanced SIMD, Element or Structure Load and Store
  def SwiftWrite2xP2FourCy : SchedWriteRes<[SwiftUnitP2]> {
      let Latency = 4;
      let ReleaseAtCycles = [2];
  }
  def SwiftWrite3xP2FourCy : SchedWriteRes<[SwiftUnitP2]> {
      let Latency = 4;
      let ReleaseAtCycles = [3];
  }
  foreach Num = 1-2 in {
    def SwiftExt#Num#xP0 : SchedWriteRes<[SwiftUnitP0]> {
      let Latency = 0;
      let NumMicroOps = Num;
      let ReleaseAtCycles = [Num];
    }
  }
  // VLDx
  // Multiple structures.
```
- EN: Defines TableGen record `SwiftWrite2xP2FourCy` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftWrite2xP2FourCy`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 878-895
```tablegen
  // Single element structure loads.
  // We assume aligned.
  // Single/two register.
  def : InstRW<[SwiftWriteLM4Cy], (instregex "VLD1(d|q)(8|16|32|64)$")>;
  def : InstRW<[SwiftWriteLM4Cy, SwiftWriteP01OneCycle],
        (instregex "VLD1(d|q)(8|16|32|64)wb")>;
  // Three register.
  def : InstRW<[SwiftWrite3xP2FourCy],
        (instregex "VLD1(d|q)(8|16|32|64)T$", "VLD1d64TPseudo")>;
  def : InstRW<[SwiftWrite3xP2FourCy, SwiftWriteP01OneCycle],
        (instregex "VLD1(d|q)(8|16|32|64)Twb")>;
  /// Four Register.
  def : InstRW<[SwiftWrite2xP2FourCy],
        (instregex "VLD1(d|q)(8|16|32|64)Q$", "VLD1d64QPseudo")>;
  def : InstRW<[SwiftWrite2xP2FourCy, SwiftWriteP01OneCycle],
        (instregex "VLD1(d|q)(8|16|32|64)Qwb")>;
  // Two element structure loads.
  // Two/four register.
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 896-906
```tablegen
  def : InstRW<[SwiftWriteLM9Cy, SwiftExt2xP0, SwiftVLDMPerm2],
        (instregex "VLD2(d|q|b)(8|16|32)$", "VLD2q(8|16|32)Pseudo$")>;
  def : InstRW<[SwiftWriteLM9Cy, SwiftWriteP01OneCycle, SwiftExt2xP0,
                SwiftVLDMPerm2],
        (instregex "VLD2(d|q|b)(8|16|32)wb", "VLD2q(8|16|32)PseudoWB")>;
  // Three element structure.
  def : InstRW<[SwiftWriteLM9Cy, SwiftWriteLM9CyNo, SwiftWriteLM9CyNo,
                SwiftVLDMPerm3, SwiftWrite3xP2FourCy],
        (instregex "VLD3(d|q)(8|16|32)$")>;
  def : InstRW<[SwiftWriteLM9Cy, SwiftVLDMPerm3, SwiftWrite3xP2FourCy],
        (instregex "VLD3(d|q)(8|16|32)(oddP|P)seudo$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 908-925
```tablegen
  def : InstRW<[SwiftWriteLM9Cy, SwiftWriteLM9CyNo, SwiftWriteLM9CyNo,
                SwiftWriteP01OneCycle, SwiftVLDMPerm3, SwiftWrite3xP2FourCy],
        (instregex "VLD3(d|q)(8|16|32)_UPD$")>;
  def : InstRW<[SwiftWriteLM9Cy, SwiftWriteP01OneCycle, SwiftVLDMPerm3,
                SwiftWrite3xP2FourCy],
        (instregex "VLD3(d|q)(8|16|32)(oddP|P)seudo_UPD")>;
  // Four element structure loads.
  def : InstRW<[SwiftWriteLM11Cy, SwiftWriteLM11Cy, SwiftWriteLM11Cy,
                SwiftWriteLM11Cy, SwiftExt2xP0, SwiftVLDMPerm4,
                SwiftWrite3xP2FourCy],
        (instregex "VLD4(d|q)(8|16|32)$")>;
  def : InstRW<[SwiftWriteLM11Cy,  SwiftExt2xP0, SwiftVLDMPerm4,
                SwiftWrite3xP2FourCy],
        (instregex "VLD4(d|q)(8|16|32)(oddP|P)seudo$")>;
  def : InstRW<[SwiftWriteLM11Cy, SwiftWriteLM11Cy, SwiftWriteLM11Cy,
                SwiftWriteLM11Cy, SwiftWriteP01OneCycle, SwiftExt2xP0,
                SwiftVLDMPerm4, SwiftWrite3xP2FourCy],
        (instregex "VLD4(d|q)(8|16|32)_UPD")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 926-928
```tablegen
  def : InstRW<[SwiftWriteLM11Cy, SwiftWriteP01OneCycle, SwiftExt2xP0,
                SwiftVLDMPerm4, SwiftWrite3xP2FourCy],
        (instregex  "VLD4(d|q)(8|16|32)(oddP|P)seudo_UPD")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 930-947
```tablegen
  // Single all/lane loads.
  // One element structure.
  def : InstRW<[SwiftWriteLM6Cy, SwiftVLDMPerm2],
        (instregex "VLD1(LN|DUP)(d|q)(8|16|32)$", "VLD1(LN|DUP)(d|q)(8|16|32)Pseudo$")>;
  def : InstRW<[SwiftWriteLM6Cy, SwiftWriteP01OneCycle, SwiftVLDMPerm2],
        (instregex "VLD1(LN|DUP)(d|q)(8|16|32)(wb|_UPD)",
                  "VLD1LNq(8|16|32)Pseudo_UPD")>;
  // Two element structure.
  def : InstRW<[SwiftWriteLM6Cy, SwiftWriteLM6Cy, SwiftExt1xP0, SwiftVLDMPerm2],
        (instregex "VLD2(DUP|LN)(d|q)(8|16|32|8x2|16x2|32x2)$",
                   "VLD2LN(d|q)(8|16|32)Pseudo$")>;
  def : InstRW<[SwiftWriteLM6Cy, SwiftWriteLM6Cy, SwiftWriteP01OneCycle,
                SwiftExt1xP0, SwiftVLDMPerm2],
        (instregex "VLD2LN(d|q)(8|16|32)_UPD$")>;
  def : InstRW<[SwiftWriteLM6Cy, SwiftWriteP01OneCycle, SwiftWriteLM6Cy,
                SwiftExt1xP0, SwiftVLDMPerm2],
        (instregex "VLD2DUPd(8|16|32|8x2|16x2|32x2)wb")>;
  def : InstRW<[SwiftWriteLM6Cy, SwiftWriteP01OneCycle, SwiftWriteLM6Cy,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 948-965
```tablegen
                SwiftExt1xP0, SwiftVLDMPerm2],
        (instregex "VLD2LN(d|q)(8|16|32)Pseudo_UPD")>;
  // Three element structure.
  def : InstRW<[SwiftWriteLM7Cy, SwiftWriteLM8Cy, SwiftWriteLM8Cy, SwiftExt1xP0,
                SwiftVLDMPerm3],
        (instregex "VLD3(DUP|LN)(d|q)(8|16|32)$",
                   "VLD3(LN|DUP)(d|q)(8|16|32)Pseudo$")>;
  def : InstRW<[SwiftWriteLM7Cy, SwiftWriteLM8Cy, SwiftWriteLM8Cy,
                SwiftWriteP01OneCycle, SwiftExt1xP0, SwiftVLDMPerm3],
        (instregex "VLD3(LN|DUP)(d|q)(8|16|32)_UPD")>;
  def : InstRW<[SwiftWriteLM7Cy, SwiftWriteP01OneCycle, SwiftWriteLM8Cy,
                SwiftWriteLM8Cy, SwiftExt1xP0, SwiftVLDMPerm3],
        (instregex "VLD3(LN|DUP)(d|q)(8|16|32)Pseudo_UPD")>;
  // Four element structure.
  def : InstRW<[SwiftWriteLM8Cy, SwiftWriteLM9Cy, SwiftWriteLM10CyNo,
                SwiftWriteLM10CyNo, SwiftExt1xP0, SwiftVLDMPerm5],
        (instregex "VLD4(LN|DUP)(d|q)(8|16|32)$",
                   "VLD4(LN|DUP)(d|q)(8|16|32)Pseudo$")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 966-983
```tablegen
  def : InstRW<[SwiftWriteLM8Cy, SwiftWriteLM9Cy, SwiftWriteLM10CyNo,
                SwiftWriteLM10CyNo, SwiftWriteP01OneCycle, SwiftExt1xP0,
                SwiftVLDMPerm5],
        (instregex "VLD4(DUP|LN)(d|q)(8|16|32)_UPD")>;
  def : InstRW<[SwiftWriteLM8Cy, SwiftWriteP01OneCycle, SwiftWriteLM9Cy,
                SwiftWriteLM10CyNo, SwiftWriteLM10CyNo, SwiftExt1xP0,
                SwiftVLDMPerm5],
        (instregex "VLD4(DUP|LN)(d|q)(8|16|32)Pseudo_UPD")>;
  // VSTx
  // Multiple structures.
  // Single element structure store.
  def : InstRW<[SwiftWrite1xP2], (instregex "VST1d(8|16|32|64)$")>;
  def : InstRW<[SwiftWrite2xP2], (instregex "VST1q(8|16|32|64)$")>;
  def : InstRW<[SwiftWriteP01OneCycle, SwiftWrite1xP2],
        (instregex "VST1d(8|16|32|64)wb")>;
  def : InstRW<[SwiftWriteP01OneCycle, SwiftWrite2xP2],
        (instregex "VST1q(8|16|32|64)wb")>;
  def : InstRW<[SwiftWrite3xP2],
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 984-1001
```tablegen
        (instregex "VST1d(8|16|32|64)T$", "VST1d64TPseudo$")>;
  def : InstRW<[SwiftWriteP01OneCycle, SwiftWrite3xP2],
        (instregex "VST1d(8|16|32|64)Twb", "VST1d64TPseudoWB")>;
  def : InstRW<[SwiftWrite4xP2],
        (instregex "VST1d(8|16|32|64)(Q|QPseudo)$")>;
  def : InstRW<[SwiftWriteP01OneCycle, SwiftWrite4xP2],
        (instregex "VST1d(8|16|32|64)(Qwb|QPseudoWB)")>;
  // Two element structure store.
  def : InstRW<[SwiftWrite1xP2, SwiftVLDMPerm1],
        (instregex "VST2(d|b)(8|16|32)$")>;
  def : InstRW<[SwiftWriteP01OneCycle, SwiftWrite1xP2, SwiftVLDMPerm1],
        (instregex "VST2(b|d)(8|16|32)wb")>;
  def : InstRW<[SwiftWrite2xP2, SwiftVLDMPerm2],
        (instregex "VST2q(8|16|32)$", "VST2q(8|16|32)Pseudo$")>;
  def : InstRW<[SwiftWrite2xP2, SwiftVLDMPerm2],
        (instregex "VST2q(8|16|32)wb", "VST2q(8|16|32)PseudoWB")>;
  // Three element structure store.
  def : InstRW<[SwiftWrite4xP2, SwiftVLDMPerm2],
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1002-1019
```tablegen
        (instregex "VST3(d|q)(8|16|32)$", "VST3(d|q)(8|16|32)(oddP|P)seudo$")>;
  def : InstRW<[SwiftWriteP01OneCycle, SwiftWrite4xP2, SwiftVLDMPerm2],
        (instregex "VST3(d|q)(8|16|32)_UPD",
                   "VST3(d|q)(8|16|32)(oddP|P)seudo_UPD$")>;
  // Four element structure store.
  def : InstRW<[SwiftWrite4xP2, SwiftVLDMPerm2],
        (instregex "VST4(d|q)(8|16|32)$", "VST4(d|q)(8|16|32)(oddP|P)seudo$")>;
  def : InstRW<[SwiftWriteP01OneCycle, SwiftWrite4xP2, SwiftVLDMPerm4],
        (instregex "VST4(d|q)(8|16|32)_UPD",
                   "VST4(d|q)(8|16|32)(oddP|P)seudo_UPD$")>;
  // Single/all lane store.
  // One element structure.
  def : InstRW<[SwiftWrite1xP2, SwiftVLDMPerm1],
        (instregex "VST1LNd(8|16|32)$", "VST1LNq(8|16|32)Pseudo$")>;
  def : InstRW<[SwiftWriteP01OneCycle, SwiftWrite1xP2, SwiftVLDMPerm1],
        (instregex "VST1LNd(8|16|32)_UPD", "VST1LNq(8|16|32)Pseudo_UPD")>;
  // Two element structure.
  def : InstRW<[SwiftWrite1xP2, SwiftVLDMPerm2],
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1020-1035
```tablegen
        (instregex "VST2LN(d|q)(8|16|32)$", "VST2LN(d|q)(8|16|32)Pseudo$")>;
  def : InstRW<[SwiftWriteP01OneCycle, SwiftWrite1xP2, SwiftVLDMPerm2],
        (instregex "VST2LN(d|q)(8|16|32)_UPD",
                   "VST2LN(d|q)(8|16|32)Pseudo_UPD")>;
  // Three element structure.
  def : InstRW<[SwiftWrite4xP2, SwiftVLDMPerm2],
        (instregex "VST3LN(d|q)(8|16|32)$", "VST3LN(d|q)(8|16|32)Pseudo$")>;
  def : InstRW<[SwiftWriteP01OneCycle, SwiftWrite4xP2, SwiftVLDMPerm2],
        (instregex "VST3LN(d|q)(8|16|32)_UPD",
                   "VST3LN(d|q)(8|16|32)Pseudo_UPD")>;
  // Four element structure.
  def : InstRW<[SwiftWrite2xP2, SwiftVLDMPerm2],
        (instregex "VST4LN(d|q)(8|16|32)$", "VST4LN(d|q)(8|16|32)Pseudo$")>;
  def : InstRW<[SwiftWriteP01OneCycle, SwiftWrite2xP2, SwiftVLDMPerm2],
        (instregex "VST4LN(d|q)(8|16|32)_UPD",
                   "VST4LN(d|q)(8|16|32)Pseudo_UPD")>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1037-1049
```tablegen
  // 4.2.44 VFP, Divide and Square Root
  def SwiftDiv17 : SchedWriteRes<[SwiftUnitP0, SwiftUnitDiv]> {
    let NumMicroOps = 1;
    let Latency = 17;
    let ReleaseAtCycles = [1, 15];
  }
  def SwiftDiv32 : SchedWriteRes<[SwiftUnitP0, SwiftUnitDiv]> {
    let NumMicroOps = 1;
    let Latency = 32;
    let ReleaseAtCycles = [1, 30];
  }
  def : InstRW<[SwiftDiv17], (instregex "VDIVS", "VSQRTS")>;
  def : InstRW<[SwiftDiv32], (instregex "VDIVD", "VSQRTD")>;
```
- EN: Defines TableGen record `SwiftDiv17` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SwiftDiv17`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1051-1055
```tablegen
  // ===---------------------------------------------------------------------===//
  // Floating-point. Map target defined SchedReadWrite to processor specific ones
  //
  def : SchedAlias<WriteFPCVT, SwiftWriteP1FourCycle>;
  def : SchedAlias<WriteFPMOV, SwiftWriteP2ThreeCycle>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1057-1058
```tablegen
  def : SchedAlias<WriteFPALU32, SwiftWriteP0FourCycle>;
  def : SchedAlias<WriteFPALU64, SwiftWriteP0SixCycle>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1060-1061
```tablegen
  def : SchedAlias<WriteFPMUL32, SwiftWriteP1FourCycle>;
  def : SchedAlias<WriteFPMUL64, SwiftWriteP1SixCycle>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1063-1064
```tablegen
  def : SchedAlias<WriteFPMAC32, SwiftWriteP1FourCycle>;
  def : SchedAlias<WriteFPMAC64, SwiftWriteP1FourCycle>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1066-1067
```tablegen
  def : SchedAlias<WriteFPDIV32, SwiftDiv17>;
  def : SchedAlias<WriteFPSQRT32, SwiftDiv17>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1069-1070
```tablegen
  def : SchedAlias<WriteFPDIV64, SwiftDiv32>;
  def : SchedAlias<WriteFPSQRT64, SwiftDiv32>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1072-1073
```tablegen
  def : ReadAdvance<ReadFPMUL, 0>;
  def : ReadAdvance<ReadFPMAC, 0>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1075-1087
```tablegen
  // Overriden via InstRW for this processor.
  def : WriteRes<WriteVLD1, []>;
  def : WriteRes<WriteVLD2, []>;
  def : WriteRes<WriteVLD3, []>;
  def : WriteRes<WriteVLD4, []>;
  def : WriteRes<WriteVST1, []>;
  def : WriteRes<WriteVST2, []>;
  def : WriteRes<WriteVST3, []>;
  def : WriteRes<WriteVST4, []>;
  // Preload.
  def : WriteRes<WritePreLd, [SwiftUnitP2]> { let Latency = 0;
    let ReleaseAtCycles = [0];
  }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1089-1089
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

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
