# MipsDSPInstrFormats.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsDSPInstrFormats.td`
- Repository: `llvm-project`
- Purpose (EN): Defines `MipsDSPInstrFormats` in LLVM TableGen DSL for the Mips backend, covering instruction encoding formats and bitfield layout.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `MipsDSPInstrFormats`，涵盖指令编码格式与位域布局。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===- MipsDSPInstrFormats.td - Mips Instruction Formats ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 9-9
```tablegen
class DspMMRel;
```
- EN: Declares reusable TableGen class `DspMMRel` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `DspMMRel`，通常用于抽象共享字段、谓词或编码结构。

### Lines 11-21
```tablegen
def Dsp2MicroMips : InstrMapping {
  let FilterClass = "DspMMRel";
  // Instructions with the same BaseOpcode and isNVStore values form a row.
  let RowFields = ["BaseOpcode"];
  // Instructions with the same predicate sense form a column.
  let ColFields = ["Arch"];
  // The key column is the unpredicated instructions.
  let KeyCol = ["dsp"];
  // Value columns are PredSense=true and PredSense=false
  let ValueCols = [["dsp"], ["mmdsp"]];
}
```
- EN: Defines TableGen record `Dsp2MicroMips` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Dsp2MicroMips`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 23-28
```tablegen
def HasDSP : Predicate<"Subtarget->hasDSP()">,
             AssemblerPredicate<(all_of FeatureDSP)>;
def HasDSPR2 : Predicate<"Subtarget->hasDSPR2()">,
               AssemblerPredicate<(all_of FeatureDSPR2)>;
def HasDSPR3 : Predicate<"Subtarget->hasDSPR3()">,
               AssemblerPredicate<(all_of FeatureDSPR3)>;
```
- EN: Defines TableGen record `HasDSP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasDSP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 30-32
```tablegen
class ISA_DSPR2 {
  list<Predicate> ASEPredicate = [HasDSPR2];
}
```
- EN: Declares reusable TableGen class `ISA_DSPR2` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `ISA_DSPR2`，通常用于抽象共享字段、谓词或编码结构。

### Lines 34-36
```tablegen
class ISA_DSPR3 {
  list<Predicate> ASEPredicate = [HasDSPR3];
}
```
- EN: Declares reusable TableGen class `ISA_DSPR3` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `ISA_DSPR3`，通常用于抽象共享字段、谓词或编码结构。

### Lines 38-41
```tablegen
// Fields.
class Field6<bits<6> val> {
  bits<6> V = val;
}
```
- EN: Declares reusable TableGen class `Field6` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `Field6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 43-44
```tablegen
def SPECIAL3_OPCODE : Field6<0b011111>;
def REGIMM_OPCODE : Field6<0b000001>;
```
- EN: Defines TableGen record `SPECIAL3_OPCODE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SPECIAL3_OPCODE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 46-51
```tablegen
class DSPInst<string opstr = "">
    : MipsInst<(outs), (ins), "", [], FrmOther> {
  let ASEPredicate = [HasDSP];
  string BaseOpcode = opstr;
  string Arch = "dsp";
}
```
- EN: Declares reusable TableGen class `DSPInst` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `DSPInst`，通常用于抽象共享字段、谓词或编码结构。

### Lines 53-56
```tablegen
class PseudoDSP<dag outs, dag ins, list<dag> pattern>
    : MipsPseudo<outs, ins, pattern> {
  let ASEPredicate = [HasDSP];
}
```
- EN: Declares reusable TableGen class `PseudoDSP` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `PseudoDSP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 58-61
```tablegen
class DSPInstAlias<string Asm, dag Result, bit Emit = 0b1>
    : InstAlias<Asm, Result, Emit>, PredicateControl {
  let ASEPredicate = [HasDSP];
}
```
- EN: Declares reusable TableGen class `DSPInstAlias` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `DSPInstAlias`，通常用于抽象共享字段、谓词或编码结构。

### Lines 63-67
```tablegen
// ADDU.QB sub-class format.
class ADDU_QB_FMT<bits<5> op> : DSPInst {
  bits<5> rd;
  bits<5> rs;
  bits<5> rt;
```
- EN: Declares reusable TableGen class `format` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `format`，通常用于抽象共享字段、谓词或编码结构。

### Lines 69-69
```tablegen
  let Opcode = SPECIAL3_OPCODE.V;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 71-76
```tablegen
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-11} = rd;
  let Inst{10-6}  = op;
  let Inst{5-0}   = 0b010000;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 78-80
```tablegen
class RADDU_W_QB_FMT<bits<5> op> : DSPInst {
  bits<5> rd;
  bits<5> rs;
```
- EN: Declares reusable TableGen class `RADDU_W_QB_FMT` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `RADDU_W_QB_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 82-82
```tablegen
  let Opcode = SPECIAL3_OPCODE.V;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 84-89
```tablegen
  let Inst{25-21} = rs;
  let Inst{20-16} = 0;
  let Inst{15-11} = rd;
  let Inst{10-6}  = op;
  let Inst{5-0}   = 0b010000;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 91-94
```tablegen
// CMPU.EQ.QB sub-class format.
class CMP_EQ_QB_R2_FMT<bits<5> op> : DSPInst {
  bits<5> rs;
  bits<5> rt;
```
- EN: Declares reusable TableGen class `format` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `format`，通常用于抽象共享字段、谓词或编码结构。

### Lines 96-96
```tablegen
  let Opcode = SPECIAL3_OPCODE.V;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 98-103
```tablegen
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-11} = 0;
  let Inst{10-6}  = op;
  let Inst{5-0}   = 0b010001;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 105-108
```tablegen
class CMP_EQ_QB_R3_FMT<bits<5> op> : DSPInst {
  bits<5> rs;
  bits<5> rt;
  bits<5> rd;
```
- EN: Declares reusable TableGen class `CMP_EQ_QB_R3_FMT` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `CMP_EQ_QB_R3_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 110-110
```tablegen
  let Opcode = SPECIAL3_OPCODE.V;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 112-117
```tablegen
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-11} = rd;
  let Inst{10-6}  = op;
  let Inst{5-0}   = 0b010001;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 119-122
```tablegen
class PRECR_SRA_PH_W_FMT<bits<5> op> : DSPInst {
  bits<5> rs;
  bits<5> rt;
  bits<5> sa;
```
- EN: Declares reusable TableGen class `PRECR_SRA_PH_W_FMT` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `PRECR_SRA_PH_W_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 124-124
```tablegen
  let Opcode = SPECIAL3_OPCODE.V;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 126-131
```tablegen
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-11} = sa;
  let Inst{10-6}  = op;
  let Inst{5-0}   = 0b010001;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 133-136
```tablegen
// ABSQ_S.PH sub-class format.
class ABSQ_S_PH_R2_FMT<bits<5> op> : DSPInst {
  bits<5> rd;
  bits<5> rt;
```
- EN: Declares reusable TableGen class `format` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `format`，通常用于抽象共享字段、谓词或编码结构。

### Lines 138-138
```tablegen
  let Opcode = SPECIAL3_OPCODE.V;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 140-145
```tablegen
  let Inst{25-21} = 0;
  let Inst{20-16} = rt;
  let Inst{15-11} = rd;
  let Inst{10-6}  = op;
  let Inst{5-0}   = 0b010010;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 148-150
```tablegen
class REPL_FMT<bits<5> op> : DSPInst {
  bits<5> rd;
  bits<10> imm;
```
- EN: Declares reusable TableGen class `REPL_FMT` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `REPL_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 152-152
```tablegen
  let Opcode = SPECIAL3_OPCODE.V;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 154-158
```tablegen
  let Inst{25-16} = imm;
  let Inst{15-11} = rd;
  let Inst{10-6}  = op;
  let Inst{5-0}   = 0b010010;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 160-164
```tablegen
// SHLL.QB sub-class format.
class SHLL_QB_FMT<bits<5> op> : DSPInst {
  bits<5> rd;
  bits<5> rt;
  bits<5> rs_sa;
```
- EN: Declares reusable TableGen class `format` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `format`，通常用于抽象共享字段、谓词或编码结构。

### Lines 166-166
```tablegen
  let Opcode = SPECIAL3_OPCODE.V;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 168-173
```tablegen
  let Inst{25-21} = rs_sa;
  let Inst{20-16} = rt;
  let Inst{15-11} = rd;
  let Inst{10-6}  = op;
  let Inst{5-0}   = 0b010011;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 175-179
```tablegen
// LX sub-class format.
class LX_FMT<bits<5> op> : DSPInst {
  bits<5> rd;
  bits<5> base;
  bits<5> index;
```
- EN: Declares reusable TableGen class `format` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `format`，通常用于抽象共享字段、谓词或编码结构。

### Lines 181-181
```tablegen
  let Opcode = SPECIAL3_OPCODE.V;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 183-188
```tablegen
  let Inst{25-21} = base;
  let Inst{20-16} = index;
  let Inst{15-11} = rd;
  let Inst{10-6}  = op;
  let Inst{5-0} = 0b001010;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 190-194
```tablegen
// ADDUH.QB sub-class format.
class ADDUH_QB_FMT<bits<5> op> : DSPInst {
  bits<5> rd;
  bits<5> rs;
  bits<5> rt;
```
- EN: Declares reusable TableGen class `format` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `format`，通常用于抽象共享字段、谓词或编码结构。

### Lines 196-196
```tablegen
  let Opcode = SPECIAL3_OPCODE.V;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 198-203
```tablegen
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-11} = rd;
  let Inst{10-6} = op;
  let Inst{5-0} = 0b011000;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 205-209
```tablegen
// APPEND sub-class format.
class APPEND_FMT<bits<5> op> : DSPInst {
  bits<5> rt;
  bits<5> rs;
  bits<5> sa;
```
- EN: Declares reusable TableGen class `format` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `format`，通常用于抽象共享字段、谓词或编码结构。

### Lines 211-211
```tablegen
  let Opcode = SPECIAL3_OPCODE.V;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 213-218
```tablegen
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-11} = sa;
  let Inst{10-6} = op;
  let Inst{5-0} = 0b110001;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 220-224
```tablegen
// DPA.W.PH sub-class format.
class DPA_W_PH_FMT<bits<5> op> : DSPInst {
  bits<2> ac;
  bits<5> rs;
  bits<5> rt;
```
- EN: Declares reusable TableGen class `format` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `format`，通常用于抽象共享字段、谓词或编码结构。

### Lines 226-226
```tablegen
  let Opcode = SPECIAL3_OPCODE.V;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 228-234
```tablegen
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-13} = 0;
  let Inst{12-11} = ac;
  let Inst{10-6}  = op;
  let Inst{5-0} = 0b110000;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 236-240
```tablegen
// MULT sub-class format.
class MULT_FMT<bits<6> opcode, bits<6> funct> : DSPInst {
  bits<2> ac;
  bits<5> rs;
  bits<5> rt;
```
- EN: Declares reusable TableGen class `format` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `format`，通常用于抽象共享字段、谓词或编码结构。

### Lines 242-242
```tablegen
  let Opcode = opcode;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 244-250
```tablegen
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-13} = 0;
  let Inst{12-11} = ac;
  let Inst{10-6}  = 0;
  let Inst{5-0} = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 252-255
```tablegen
// MFHI sub-class format.
class MFHI_FMT<bits<6> funct> : DSPInst {
  bits<5> rd;
  bits<2> ac;
```
- EN: Declares reusable TableGen class `format` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `format`，通常用于抽象共享字段、谓词或编码结构。

### Lines 257-264
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-23} = 0;
  let Inst{22-21} = ac;
  let Inst{20-16} = 0;
  let Inst{15-11} = rd;
  let Inst{10-6} = 0;
  let Inst{5-0} = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 266-269
```tablegen
// MTHI sub-class format.
class MTHI_FMT<bits<6> funct> : DSPInst {
  bits<5> rs;
  bits<2> ac;
```
- EN: Declares reusable TableGen class `format` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `format`，通常用于抽象共享字段、谓词或编码结构。

### Lines 271-277
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-21} = rs;
  let Inst{20-13} = 0;
  let Inst{12-11} = ac;
  let Inst{10-6} = 0;
  let Inst{5-0} = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 279-283
```tablegen
// EXTR.W sub-class format (type 1).
class EXTR_W_TY1_FMT<bits<5> op> : DSPInst {
  bits<5> rt;
  bits<2> ac;
  bits<5> shift_rs;
```
- EN: Declares reusable TableGen class `format` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `format`，通常用于抽象共享字段、谓词或编码结构。

### Lines 285-285
```tablegen
  let Opcode = SPECIAL3_OPCODE.V;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 287-293
```tablegen
  let Inst{25-21} = shift_rs;
  let Inst{20-16} = rt;
  let Inst{15-13} = 0;
  let Inst{12-11} = ac;
  let Inst{10-6} = op;
  let Inst{5-0} = 0b111000;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 295-298
```tablegen
// SHILO sub-class format.
class SHILO_R1_FMT<bits<5> op> : DSPInst {
  bits<2> ac;
  bits<6> shift;
```
- EN: Declares reusable TableGen class `format` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `format`，通常用于抽象共享字段、谓词或编码结构。

### Lines 300-300
```tablegen
  let Opcode = SPECIAL3_OPCODE.V;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 302-307
```tablegen
  let Inst{25-20} = shift;
  let Inst{19-13} = 0;
  let Inst{12-11} = ac;
  let Inst{10-6} = op;
  let Inst{5-0} = 0b111000;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 309-311
```tablegen
class SHILO_R2_FMT<bits<5> op> : DSPInst {
  bits<2> ac;
  bits<5> rs;
```
- EN: Declares reusable TableGen class `SHILO_R2_FMT` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `SHILO_R2_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 313-313
```tablegen
  let Opcode = SPECIAL3_OPCODE.V;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 315-320
```tablegen
  let Inst{25-21} = rs;
  let Inst{20-13} = 0;
  let Inst{12-11} = ac;
  let Inst{10-6} = op;
  let Inst{5-0} = 0b111000;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 322-324
```tablegen
class RDDSP_FMT<bits<5> op> : DSPInst {
  bits<5> rd;
  bits<10> mask;
```
- EN: Declares reusable TableGen class `RDDSP_FMT` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `RDDSP_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 326-326
```tablegen
  let Opcode = SPECIAL3_OPCODE.V;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 328-332
```tablegen
  let Inst{25-16} = mask;
  let Inst{15-11} = rd;
  let Inst{10-6} = op;
  let Inst{5-0} = 0b111000;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 334-336
```tablegen
class WRDSP_FMT<bits<5> op> : DSPInst {
  bits<5> rs;
  bits<10> mask;
```
- EN: Declares reusable TableGen class `WRDSP_FMT` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `WRDSP_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 338-338
```tablegen
  let Opcode = SPECIAL3_OPCODE.V;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 340-344
```tablegen
  let Inst{25-21} = rs;
  let Inst{20-11} = mask;
  let Inst{10-6} = op;
  let Inst{5-0} = 0b111000;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 346-347
```tablegen
class BPOSGE32_FMT<bits<5> op> : DSPInst {
  bits<16> offset;
```
- EN: Declares reusable TableGen class `BPOSGE32_FMT` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `BPOSGE32_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 349-349
```tablegen
  let Opcode = REGIMM_OPCODE.V;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 351-354
```tablegen
  let Inst{25-21} = 0;
  let Inst{20-16} = op;
  let Inst{15-0} = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 356-359
```tablegen
// INSV sub-class format.
class INSV_FMT<bits<6> op> : DSPInst {
  bits<5> rt;
  bits<5> rs;
```
- EN: Declares reusable TableGen class `format` for `MipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsDSPInstrFormats` 声明可复用的 TableGen 类 `format`，通常用于抽象共享字段、谓词或编码结构。

### Lines 361-361
```tablegen
  let Opcode = SPECIAL3_OPCODE.V;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 363-367
```tablegen
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-6} = 0;
  let Inst{5-0} = op;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

## Key Concepts / 关键概念

- EN: Primary role: instruction encoding formats and bitfield layout.
  - CN: 核心职责：指令编码格式与位域布局。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。
- EN: Bitfield assignments describe exact instruction encodings and per-instruction flags.
  - CN: 位字段赋值用于描述精确的指令编码以及每条指令的标志位。
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `(none)`.
  - CN: TableGen 包含项：`(none)`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
