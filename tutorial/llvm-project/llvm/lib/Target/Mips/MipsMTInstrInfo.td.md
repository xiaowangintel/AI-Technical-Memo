# MipsMTInstrInfo.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsMTInstrInfo.td`
- Repository: `llvm-project`
- Purpose (EN): This file describes the MIPS MT ASE as defined by MD00378 1.12.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `MipsMTInstrInfo`，涵盖指令语义、调度提示以及机器级辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```tablegen
//===-- MipsMTInstrInfo.td - Mips MT Instruction Infos -----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the MIPS MT ASE as defined by MD00378 1.12.
//
// TODO: Add support for the microMIPS encodings for the MT ASE and add the
//       instruction mappings.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 16-18
```tablegen
//===----------------------------------------------------------------------===//
// MIPS MT Instruction Encodings
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 20-21
```tablegen
class DMT_ENC : COP0_MFMC0_MT<FIELD5_1_DMT_EMT, FIELD5_2_DMT_EMT,
                              OPCODE_SC_D>;
```
- EN: Declares reusable TableGen class `DMT_ENC` for `MipsMTInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMTInstrInfo` 声明可复用的 TableGen 类 `DMT_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 23-24
```tablegen
class EMT_ENC : COP0_MFMC0_MT<FIELD5_1_DMT_EMT, FIELD5_2_DMT_EMT,
                              OPCODE_SC_E>;
```
- EN: Declares reusable TableGen class `EMT_ENC` for `MipsMTInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMTInstrInfo` 声明可复用的 TableGen 类 `EMT_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 26-27
```tablegen
class DVPE_ENC : COP0_MFMC0_MT<FIELD5_1_2_DVPE_EVPE, FIELD5_1_2_DVPE_EVPE,
                               OPCODE_SC_D>;
```
- EN: Declares reusable TableGen class `DVPE_ENC` for `MipsMTInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMTInstrInfo` 声明可复用的 TableGen 类 `DVPE_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 29-30
```tablegen
class EVPE_ENC : COP0_MFMC0_MT<FIELD5_1_2_DVPE_EVPE, FIELD5_1_2_DVPE_EVPE,
                               OPCODE_SC_E>;
```
- EN: Declares reusable TableGen class `EVPE_ENC` for `MipsMTInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMTInstrInfo` 声明可复用的 TableGen 类 `EVPE_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 32-32
```tablegen
class FORK_ENC : SPECIAL3_MT_FORK;
```
- EN: Declares reusable TableGen class `FORK_ENC` for `MipsMTInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMTInstrInfo` 声明可复用的 TableGen 类 `FORK_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 34-34
```tablegen
class YIELD_ENC : SPECIAL3_MT_YIELD;
```
- EN: Declares reusable TableGen class `YIELD_ENC` for `MipsMTInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMTInstrInfo` 声明可复用的 TableGen 类 `YIELD_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 36-36
```tablegen
class MFTR_ENC : COP0_MFTTR_MT<FIELD5_MFTR>;
```
- EN: Declares reusable TableGen class `MFTR_ENC` for `MipsMTInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMTInstrInfo` 声明可复用的 TableGen 类 `MFTR_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 38-38
```tablegen
class MTTR_ENC : COP0_MFTTR_MT<FIELD5_MTTR>;
```
- EN: Declares reusable TableGen class `MTTR_ENC` for `MipsMTInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMTInstrInfo` 声明可复用的 TableGen 类 `MTTR_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 40-42
```tablegen
//===----------------------------------------------------------------------===//
// MIPS MT Instruction Descriptions
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 44-49
```tablegen
class MT_1R_DESC_BASE<string instr_asm> {
  dag OutOperandList = (outs GPR32Opnd:$rt);
  dag InOperandList = (ins);
  string AsmString = !strconcat(instr_asm, "\t$rt");
  list<dag> Pattern = [];
}
```
- EN: Declares reusable TableGen class `MT_1R_DESC_BASE` for `MipsMTInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMTInstrInfo` 声明可复用的 TableGen 类 `MT_1R_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 51-56
```tablegen
class MFTR_DESC {
  dag OutOperandList = (outs GPR32Opnd:$rd);
  dag InOperandList = (ins GPR32Opnd:$rt, uimm1:$u, uimm3:$sel, uimm1:$h);
  string AsmString = "mftr\t$rd, $rt, $u, $sel, $h";
  list<dag> Pattern = [];
}
```
- EN: Declares reusable TableGen class `MFTR_DESC` for `MipsMTInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMTInstrInfo` 声明可复用的 TableGen 类 `MFTR_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 58-63
```tablegen
class MTTR_DESC {
  dag OutOperandList = (outs GPR32Opnd:$rd);
  dag InOperandList = (ins GPR32Opnd:$rt, uimm1:$u, uimm3:$sel, uimm1:$h);
  string AsmString = "mttr\t$rt, $rd, $u, $sel, $h";
  list<dag> Pattern = [];
}
```
- EN: Declares reusable TableGen class `MTTR_DESC` for `MipsMTInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMTInstrInfo` 声明可复用的 TableGen 类 `MTTR_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 65-70
```tablegen
class FORK_DESC {
  dag OutOperandList = (outs GPR32Opnd:$rs, GPR32Opnd:$rd);
  dag InOperandList = (ins GPR32Opnd:$rt);
  string AsmString = "fork\t$rd, $rs, $rt";
  list<dag> Pattern = [];
}
```
- EN: Declares reusable TableGen class `FORK_DESC` for `MipsMTInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMTInstrInfo` 声明可复用的 TableGen 类 `FORK_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 72-77
```tablegen
class YIELD_DESC {
  dag OutOperandList = (outs GPR32Opnd:$rd);
  dag InOperandList = (ins GPR32Opnd:$rs);
  string AsmString = "yield\t$rd, $rs";
  list<dag> Pattern = [];
}
```
- EN: Declares reusable TableGen class `YIELD_DESC` for `MipsMTInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMTInstrInfo` 声明可复用的 TableGen 类 `YIELD_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 79-79
```tablegen
class DMT_DESC : MT_1R_DESC_BASE<"dmt">;
```
- EN: Declares reusable TableGen class `DMT_DESC` for `MipsMTInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMTInstrInfo` 声明可复用的 TableGen 类 `DMT_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 81-81
```tablegen
class EMT_DESC : MT_1R_DESC_BASE<"emt">;
```
- EN: Declares reusable TableGen class `EMT_DESC` for `MipsMTInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMTInstrInfo` 声明可复用的 TableGen 类 `EMT_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 83-83
```tablegen
class DVPE_DESC : MT_1R_DESC_BASE<"dvpe">;
```
- EN: Declares reusable TableGen class `DVPE_DESC` for `MipsMTInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMTInstrInfo` 声明可复用的 TableGen 类 `DVPE_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 85-85
```tablegen
class EVPE_DESC : MT_1R_DESC_BASE<"evpe">;
```
- EN: Declares reusable TableGen class `EVPE_DESC` for `MipsMTInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMTInstrInfo` 声明可复用的 TableGen 类 `EVPE_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 87-92
```tablegen
//===----------------------------------------------------------------------===//
// MIPS MT Instruction Definitions
//===----------------------------------------------------------------------===//
let hasSideEffects = 1, isNotDuplicable = 1,
    AdditionalPredicates = [NotInMicroMips] in {
  def DMT : DMT_ENC, DMT_DESC, ASE_MT;
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 94-94
```tablegen
  def EMT : EMT_ENC, EMT_DESC, ASE_MT;
```
- EN: Defines TableGen record `EMT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `EMT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 96-96
```tablegen
  def DVPE : DVPE_ENC, DVPE_DESC, ASE_MT;
```
- EN: Defines TableGen record `DVPE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DVPE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 98-98
```tablegen
  def EVPE : EVPE_ENC, EVPE_DESC, ASE_MT;
```
- EN: Defines TableGen record `EVPE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `EVPE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 100-100
```tablegen
  def FORK : FORK_ENC, FORK_DESC, ASE_MT;
```
- EN: Defines TableGen record `FORK` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FORK`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 102-102
```tablegen
  def YIELD : YIELD_ENC, YIELD_DESC, ASE_MT;
```
- EN: Defines TableGen record `YIELD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `YIELD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 104-104
```tablegen
  def MFTR : MFTR_ENC, MFTR_DESC, ASE_MT;
```
- EN: Defines TableGen record `MFTR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MFTR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 106-107
```tablegen
  def MTTR : MTTR_ENC, MTTR_DESC, ASE_MT;
}
```
- EN: Defines TableGen record `MTTR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MTTR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 109-114
```tablegen
//===----------------------------------------------------------------------===//
// MIPS MT Pseudo Instructions - used to support mtfr & mttr aliases.
//===----------------------------------------------------------------------===//
def MFTC0 : MipsAsmPseudoInst<(outs GPR32Opnd:$rd), (ins COP0Opnd:$rt,
                                                        uimm3:$sel),
                              "mftc0 $rd, $rt, $sel">, ASE_MT;
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 116-118
```tablegen
def MFTGPR : MipsAsmPseudoInst<(outs GPR32Opnd:$rd), (ins GPR32Opnd:$rt,
                                                          uimm3:$sel),
                               "mftgpr $rd, $rt">, ASE_MT;
```
- EN: Defines TableGen record `MFTGPR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MFTGPR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 120-121
```tablegen
def MFTLO : MipsAsmPseudoInst<(outs GPR32Opnd:$rt), (ins ACC64DSPOpnd:$ac),
                              "mftlo $rt, $ac">, ASE_MT;
```
- EN: Defines TableGen record `MFTLO` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MFTLO`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 123-124
```tablegen
def MFTHI : MipsAsmPseudoInst<(outs GPR32Opnd:$rt), (ins ACC64DSPOpnd:$ac),
                              "mfthi $rt, $ac">, ASE_MT;
```
- EN: Defines TableGen record `MFTHI` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MFTHI`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 126-127
```tablegen
def MFTACX : MipsAsmPseudoInst<(outs GPR32Opnd:$rt), (ins ACC64DSPOpnd:$ac),
                               "mftacx $rt, $ac">, ASE_MT;
```
- EN: Defines TableGen record `MFTACX` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MFTACX`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 129-130
```tablegen
def MFTDSP : MipsAsmPseudoInst<(outs GPR32Opnd:$rt), (ins),
                               "mftdsp $rt">, ASE_MT;
```
- EN: Defines TableGen record `MFTDSP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MFTDSP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 132-133
```tablegen
def MFTC1 : MipsAsmPseudoInst<(outs GPR32Opnd:$rt), (ins FGR32Opnd:$ft),
                              "mftc1 $rt, $ft">, ASE_MT;
```
- EN: Defines TableGen record `MFTC1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MFTC1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 135-136
```tablegen
def MFTHC1 : MipsAsmPseudoInst<(outs GPR32Opnd:$rt), (ins FGR32Opnd:$ft),
                               "mfthc1 $rt, $ft">, ASE_MT;
```
- EN: Defines TableGen record `MFTHC1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MFTHC1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 138-139
```tablegen
def CFTC1 : MipsAsmPseudoInst<(outs GPR32Opnd:$rt), (ins FGR32CCOpnd:$ft),
                              "cftc1 $rt, $ft">, ASE_MT;
```
- EN: Defines TableGen record `CFTC1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CFTC1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 142-144
```tablegen
def MTTC0 : MipsAsmPseudoInst<(outs COP0Opnd:$rd), (ins GPR32Opnd:$rt,
                                                        uimm3:$sel),
                              "mttc0 $rt, $rd, $sel">, ASE_MT;
```
- EN: Defines TableGen record `MTTC0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MTTC0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 146-147
```tablegen
def MTTGPR : MipsAsmPseudoInst<(outs GPR32Opnd:$rt), (ins GPR32Opnd:$rd),
                               "mttgpr $rd, $rt">, ASE_MT;
```
- EN: Defines TableGen record `MTTGPR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MTTGPR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 149-150
```tablegen
def MTTLO : MipsAsmPseudoInst<(outs ACC64DSPOpnd:$ac), (ins GPR32Opnd:$rt),
                              "mttlo $rt, $ac">, ASE_MT;
```
- EN: Defines TableGen record `MTTLO` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MTTLO`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 152-153
```tablegen
def MTTHI : MipsAsmPseudoInst<(outs ACC64DSPOpnd:$ac), (ins GPR32Opnd:$rt),
                              "mtthi $rt, $ac">, ASE_MT;
```
- EN: Defines TableGen record `MTTHI` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MTTHI`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 155-156
```tablegen
def MTTACX : MipsAsmPseudoInst<(outs ACC64DSPOpnd:$ac), (ins GPR32Opnd:$rt),
                               "mttacx $rt, $ac">, ASE_MT;
```
- EN: Defines TableGen record `MTTACX` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MTTACX`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 158-159
```tablegen
def MTTDSP : MipsAsmPseudoInst<(outs), (ins GPR32Opnd:$rt),
                               "mttdsp $rt">, ASE_MT;
```
- EN: Defines TableGen record `MTTDSP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MTTDSP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 161-162
```tablegen
def MTTC1 : MipsAsmPseudoInst<(outs FGR32Opnd:$ft), (ins GPR32Opnd:$rt),
                              "mttc1 $rt, $ft">, ASE_MT;
```
- EN: Defines TableGen record `MTTC1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MTTC1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 164-165
```tablegen
def MTTHC1 : MipsAsmPseudoInst<(outs FGR32Opnd:$ft), (ins GPR32Opnd:$rt),
                               "mtthc1 $rt, $ft">, ASE_MT;
```
- EN: Defines TableGen record `MTTHC1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MTTHC1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 167-168
```tablegen
def CTTC1 : MipsAsmPseudoInst<(outs FGR32CCOpnd:$ft), (ins GPR32Opnd:$rt),
                              "cttc1 $rt, $ft">, ASE_MT;
```
- EN: Defines TableGen record `CTTC1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CTTC1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 170-172
```tablegen
//===----------------------------------------------------------------------===//
// MIPS MT Instruction Definitions
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 174-175
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  def : MipsInstAlias<"dmt", (DMT ZERO), 1>, ASE_MT;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 177-177
```tablegen
  def : MipsInstAlias<"emt", (EMT ZERO), 1>, ASE_MT;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 179-179
```tablegen
  def : MipsInstAlias<"dvpe", (DVPE ZERO), 1>, ASE_MT;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 181-181
```tablegen
  def : MipsInstAlias<"evpe", (EVPE ZERO), 1>, ASE_MT;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 183-183
```tablegen
  def : MipsInstAlias<"yield $rs", (YIELD ZERO, GPR32Opnd:$rs), 1>, ASE_MT;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 185-186
```tablegen
  def : MipsInstAlias<"mftc0 $rd, $rt", (MFTC0 GPR32Opnd:$rd, COP0Opnd:$rt, 0),
                      1>, ASE_MT;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 188-188
```tablegen
  def : MipsInstAlias<"mftlo $rt", (MFTLO GPR32Opnd:$rt, AC0), 1>, ASE_MT;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 190-190
```tablegen
  def : MipsInstAlias<"mfthi $rt", (MFTHI GPR32Opnd:$rt, AC0), 1>, ASE_MT;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 192-192
```tablegen
  def : MipsInstAlias<"mftacx $rt", (MFTACX GPR32Opnd:$rt, AC0), 1>, ASE_MT;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 194-195
```tablegen
  def : MipsInstAlias<"mttc0 $rd, $rt", (MTTC0 COP0Opnd:$rt, GPR32Opnd:$rd, 0),
                      1>, ASE_MT;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 197-197
```tablegen
  def : MipsInstAlias<"mttlo $rt", (MTTLO AC0, GPR32Opnd:$rt), 1>, ASE_MT;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 199-199
```tablegen
  def : MipsInstAlias<"mtthi $rt", (MTTHI AC0, GPR32Opnd:$rt), 1>, ASE_MT;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 201-202
```tablegen
  def : MipsInstAlias<"mttacx $rt", (MTTACX AC0, GPR32Opnd:$rt), 1>, ASE_MT;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

## Key Concepts / 关键概念

- EN: Primary role: instruction semantics, scheduling hints, and machine-level helpers.
  - CN: 核心职责：指令语义、调度提示以及机器级辅助逻辑。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。
- EN: SelectionDAG patterns or node profiles connect IR-level intent to target instructions.
  - CN: SelectionDAG 模式或节点轮廓把 IR 层意图连接到目标指令。
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `(none)`.
  - CN: TableGen 包含项：`(none)`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
