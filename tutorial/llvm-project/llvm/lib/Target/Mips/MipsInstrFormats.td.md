# MipsInstrFormats.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsInstrFormats.td`
- Repository: `llvm-project`
- Purpose (EN): Describe MIPS instructions format CPU INSTRUCTION FORMATS opcode  - operation code.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `MipsInstrFormats`，涵盖指令编码格式与位域布局。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===-- MipsInstrFormats.td - Mips Instruction Formats -----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 9-21
```tablegen
//===----------------------------------------------------------------------===//
//  Describe MIPS instructions format
//
//  CPU INSTRUCTION FORMATS
//
//  opcode  - operation code.
//  rs      - src reg.
//  rt      - dst reg (on a 2 regs instr) or src reg (on a 3 reg instr).
//  rd      - dst reg, only used on 3 regs instr.
//  shamt   - only used on shift instructions, contains the shift amount.
//  funct   - combined with opcode field give us an operation code.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 23-28
```tablegen
// Format specifies the encoding used by the instruction.  This is part of the
// ad-hoc solution used to emit machine instruction encodings by our machine
// code emitter.
class Format<bits<4> val> {
  bits<4> Value = val;
}
```
- EN: Declares reusable TableGen class `Format` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `Format`，通常用于抽象共享字段、谓词或编码结构。

### Lines 30-36
```tablegen
def Pseudo    : Format<0>;
def FrmR      : Format<1>;
def FrmI      : Format<2>;
def FrmJ      : Format<3>;
def FrmFR     : Format<4>;
def FrmFI     : Format<5>;
def FrmOther  : Format<6>; // Instruction w/ a custom format
```
- EN: Defines TableGen record `Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 38-38
```tablegen
class MMRel;
```
- EN: Declares reusable TableGen class `MMRel` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `MMRel`，通常用于抽象共享字段、谓词或编码结构。

### Lines 40-50
```tablegen
def Std2MicroMips : InstrMapping {
  let FilterClass = "MMRel";
  // Instructions with the same BaseOpcode and isNVStore values form a row.
  let RowFields = ["BaseOpcode"];
  // Instructions with the same predicate sense form a column.
  let ColFields = ["Arch"];
  // The key column is the unpredicated instructions.
  let KeyCol = ["se"];
  // Value columns are PredSense=true and PredSense=false
  let ValueCols = [["se"], ["micromips"]];
}
```
- EN: Defines TableGen record `Std2MicroMips` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Std2MicroMips`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 52-52
```tablegen
class StdMMR6Rel;
```
- EN: Declares reusable TableGen class `StdMMR6Rel` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `StdMMR6Rel`，通常用于抽象共享字段、谓词或编码结构。

### Lines 54-64
```tablegen
def Std2MicroMipsR6 : InstrMapping {
  let FilterClass = "StdMMR6Rel";
  // Instructions with the same BaseOpcode and isNVStore values form a row.
  let RowFields = ["BaseOpcode"];
  // Instructions with the same predicate sense form a column.
  let ColFields = ["Arch"];
  // The key column is the unpredicated instructions.
  let KeyCol = ["se"];
  // Value columns are PredSense=true and PredSense=false
  let ValueCols = [["se"], ["micromipsr6"]];
}
```
- EN: Defines TableGen record `Std2MicroMipsR6` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Std2MicroMipsR6`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 66-68
```tablegen
class StdArch {
  string Arch = "se";
}
```
- EN: Declares reusable TableGen class `StdArch` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `StdArch`，通常用于抽象共享字段、谓词或编码结构。

### Lines 70-75
```tablegen
// Generic Mips Format
class MipsInst<dag outs, dag ins, string asmstr, list<dag> pattern,
               Format f>: Instruction, PredicateControl
{
  field bits<32> Inst;
  Format Form = f;
```
- EN: Declares reusable TableGen class `MipsInst` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `MipsInst`，通常用于抽象共享字段、谓词或编码结构。

### Lines 77-77
```tablegen
  let Namespace = "Mips";
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 79-79
```tablegen
  let Size = 4;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 81-81
```tablegen
  bits<6> Opcode = 0;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 83-84
```tablegen
  // Top 6 bits are the 'opcode' field
  let Inst{31-26} = Opcode;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 86-87
```tablegen
  let OutOperandList = outs;
  let InOperandList  = ins;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 89-90
```tablegen
  let AsmString   = asmstr;
  let Pattern     = pattern;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 93-101
```tablegen
  //
  // Attributes specific to Mips instructions...
  //
  bits<4> FormBits     = Form.Value;
  bit isCTI            = 0; // Any form of Control Transfer Instruction.
                            // Required for MIPSR6
  bit hasForbiddenSlot = 0; // Instruction has a forbidden slot.
  bit hasFCCRegOperand = 0; // Instruction uses $fcc<X> register and is
                            // present in MIPS-I to MIPS-III.
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 103-107
```tablegen
  // TSFlags layout should be kept in sync with MCTargetDesc/MipsBaseInfo.h.
  let TSFlags{3-0}   = FormBits;
  let TSFlags{4}     = isCTI;
  let TSFlags{5}     = hasForbiddenSlot;
  let TSFlags{6}     = hasFCCRegOperand;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 109-110
```tablegen
  let DecoderNamespace = "Mips";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 112-119
```tablegen
// Mips32/64 Instruction Format
class InstSE<dag outs, dag ins, string asmstr, list<dag> pattern,
             Format f, string opstr = ""> :
  MipsInst<outs, ins, asmstr, pattern, f> {
  let EncodingPredicates = [NotInMips16Mode];
  string BaseOpcode = opstr;
  string Arch;
}
```
- EN: Declares reusable TableGen class `InstSE` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `InstSE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 121-126
```tablegen
// Mips Pseudo Instructions Format
class MipsPseudo<dag outs, dag ins, list<dag> pattern> :
  MipsInst<outs, ins, "", pattern, Pseudo> {
  let isCodeGenOnly = 1;
  let isPseudo = 1;
}
```
- EN: Declares reusable TableGen class `MipsPseudo` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `MipsPseudo`，通常用于抽象共享字段、谓词或编码结构。

### Lines 128-132
```tablegen
// Mips32/64 Pseudo Instruction Format
class PseudoSE<dag outs, dag ins, list<dag> pattern> :
  MipsPseudo<outs, ins, pattern> {
  let EncodingPredicates = [NotInMips16Mode];
}
```
- EN: Declares reusable TableGen class `PseudoSE` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `PseudoSE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 134-145
```tablegen
// Pseudo-instructions for alternate assembly syntax (never used by codegen).
// These are aliases that require C++ handling to convert to the target
// instruction, while InstAliases can be handled directly by tblgen.
class MipsAsmPseudoInst<dag outs, dag ins, string asmstr>:
  MipsInst<outs, ins, asmstr, [], Pseudo> {
  let isPseudo = 1;
  let hasNoSchedulingInfo = 1;
  let Pattern = [];
}
//===----------------------------------------------------------------------===//
// Format R instruction class in Mips : <|opcode|rs|rt|rd|shamt|funct|>
//===----------------------------------------------------------------------===//
```
- EN: Declares reusable TableGen class `MipsAsmPseudoInst` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `MipsAsmPseudoInst`，通常用于抽象共享字段、谓词或编码结构。

### Lines 147-155
```tablegen
class FR<bits<6> op, bits<6> _funct, dag outs, dag ins, string asmstr,
         list<dag> pattern>:
  InstSE<outs, ins, asmstr, pattern, FrmR>
{
  bits<5>  rd;
  bits<5>  rs;
  bits<5>  rt;
  bits<5>  shamt;
  bits<6>  funct;
```
- EN: Declares reusable TableGen class `FR` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `FR`，通常用于抽象共享字段、谓词或编码结构。

### Lines 157-158
```tablegen
  let Opcode = op;
  let funct  = _funct;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 160-165
```tablegen
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-11} = rd;
  let Inst{10-6}  = shamt;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 167-169
```tablegen
//===----------------------------------------------------------------------===//
// Format J instruction class in Mips : <|opcode|address|>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 171-173
```tablegen
class FJ<bits<6> op> : StdArch
{
  bits<26> target;
```
- EN: Declares reusable TableGen class `FJ` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `FJ`，通常用于抽象共享字段、谓词或编码结构。

### Lines 175-175
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 177-179
```tablegen
  let Inst{31-26} = op;
  let Inst{25-0}  = target;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 181-187
```tablegen
//===----------------------------------------------------------------------===//
// MFC instruction class in Mips : <|op|mf|rt|rd|gst|0000|sel|>
//===----------------------------------------------------------------------===//
class MFC3OP_FM<bits<6> op, bits<5> mfmt, bits<3> guest> : StdArch {
  bits<5> rt;
  bits<5> rd;
  bits<3> sel;
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 189-189
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 191-198
```tablegen
  let Inst{31-26} = op;
  let Inst{25-21} = mfmt;
  let Inst{20-16} = rt;
  let Inst{15-11} = rd;
  let Inst{10-8}  = guest;
  let Inst{7-3}   = 0;
  let Inst{2-0}   = sel;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 200-202
```tablegen
class MFC2OP_FM<bits<6> op, bits<5> mfmt> : StdArch {
  bits<5>  rt;
  bits<16> imm16;
```
- EN: Declares reusable TableGen class `MFC2OP_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `MFC2OP_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 204-204
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 206-210
```tablegen
  let Inst{31-26} = op;
  let Inst{25-21} = mfmt;
  let Inst{20-16} = rt;
  let Inst{15-0}  = imm16;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 212-215
```tablegen
class ADD_FM<bits<6> op, bits<6> funct> : StdArch {
  bits<5> rd;
  bits<5> rs;
  bits<5> rt;
```
- EN: Declares reusable TableGen class `ADD_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `ADD_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 217-217
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 219-225
```tablegen
  let Inst{31-26} = op;
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-11} = rd;
  let Inst{10-6}  = 0;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 227-230
```tablegen
class ADDI_FM<bits<6> op> : StdArch {
  bits<5>  rs;
  bits<5>  rt;
  bits<16> imm16;
```
- EN: Declares reusable TableGen class `ADDI_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `ADDI_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 232-232
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 234-238
```tablegen
  let Inst{31-26} = op;
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-0}  = imm16;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 240-243
```tablegen
class SRA_FM<bits<6> funct, bit rotate> : StdArch {
  bits<5> rd;
  bits<5> rt;
  bits<5> shamt;
```
- EN: Declares reusable TableGen class `SRA_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `SRA_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 245-245
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 247-254
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-22} = 0;
  let Inst{21}    = rotate;
  let Inst{20-16} = rt;
  let Inst{15-11} = rd;
  let Inst{10-6}  = shamt;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 256-259
```tablegen
class SRLV_FM<bits<6> funct, bit rotate> : StdArch {
  bits<5> rd;
  bits<5> rt;
  bits<5> rs;
```
- EN: Declares reusable TableGen class `SRLV_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `SRLV_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 261-261
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 263-270
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-11} = rd;
  let Inst{10-7}  = 0;
  let Inst{6}     = rotate;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 272-275
```tablegen
class BEQ_FM<bits<6> op> : StdArch {
  bits<5>  rs;
  bits<5>  rt;
  bits<16> offset;
```
- EN: Declares reusable TableGen class `BEQ_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `BEQ_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 277-277
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 279-283
```tablegen
  let Inst{31-26} = op;
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 285-287
```tablegen
class BGEZ_FM<bits<6> op, bits<5> funct> : StdArch {
  bits<5>  rs;
  bits<16> offset;
```
- EN: Declares reusable TableGen class `BGEZ_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `BGEZ_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 289-289
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 291-295
```tablegen
  let Inst{31-26} = op;
  let Inst{25-21} = rs;
  let Inst{20-16} = funct;
  let Inst{15-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 297-300
```tablegen
class BBIT_FM<bits<6> op> : StdArch {
  bits<5>  rs;
  bits<5>  p;
  bits<16> offset;
```
- EN: Declares reusable TableGen class `BBIT_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `BBIT_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 302-302
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 304-308
```tablegen
  let Inst{31-26} = op;
  let Inst{25-21} = rs;
  let Inst{20-16} = p;
  let Inst{15-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 310-313
```tablegen
class SLTI_FM<bits<6> op> : StdArch {
  bits<5> rt;
  bits<5> rs;
  bits<16> imm16;
```
- EN: Declares reusable TableGen class `SLTI_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `SLTI_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 315-315
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 317-321
```tablegen
  let Inst{31-26} = op;
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-0}  = imm16;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 323-324
```tablegen
class MFLO_FM<bits<6> funct> : StdArch {
  bits<5> rd;
```
- EN: Declares reusable TableGen class `MFLO_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `MFLO_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 326-326
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 328-333
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-16} = 0;
  let Inst{15-11} = rd;
  let Inst{10-6}  = 0;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 335-336
```tablegen
class MTLO_FM<bits<6> funct> : StdArch {
  bits<5> rs;
```
- EN: Declares reusable TableGen class `MTLO_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `MTLO_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 338-338
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 340-344
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-21} = rs;
  let Inst{20-6}  = 0;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 346-348
```tablegen
class SEB_FM<bits<5> funct, bits<6> funct2> : StdArch {
  bits<5> rd;
  bits<5> rt;
```
- EN: Declares reusable TableGen class `SEB_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `SEB_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 350-350
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 352-358
```tablegen
  let Inst{31-26} = 0x1f;
  let Inst{25-21} = 0;
  let Inst{20-16} = rt;
  let Inst{15-11} = rd;
  let Inst{10-6}  = funct;
  let Inst{5-0}   = funct2;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 360-363
```tablegen
class CLO_FM<bits<6> funct> : StdArch {
  bits<5> rd;
  bits<5> rs;
  bits<5> rt;
```
- EN: Declares reusable TableGen class `CLO_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `CLO_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 365-365
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 367-374
```tablegen
  let Inst{31-26} = 0x1c;
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-11} = rd;
  let Inst{10-6}  = 0;
  let Inst{5-0}   = funct;
  let rt = rd;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 376-378
```tablegen
class LUI_FM : StdArch {
  bits<5> rt;
  bits<16> imm16;
```
- EN: Declares reusable TableGen class `LUI_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `LUI_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 380-380
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 382-386
```tablegen
  let Inst{31-26} = 0xf;
  let Inst{25-21} = 0;
  let Inst{20-16} = rt;
  let Inst{15-0}  = imm16;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 388-390
```tablegen
class JALR_FM {
  bits<5> rd;
  bits<5> rs;
```
- EN: Declares reusable TableGen class `JALR_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `JALR_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 392-392
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 394-400
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-21} = rs;
  let Inst{20-16} = 0;
  let Inst{15-11} = rd;
  let Inst{10-6}  = 0;
  let Inst{5-0}   = 9;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 402-404
```tablegen
class BGEZAL_FM<bits<5> funct> : StdArch {
  bits<5>  rs;
  bits<16> offset;
```
- EN: Declares reusable TableGen class `BGEZAL_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `BGEZAL_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 406-406
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 408-412
```tablegen
  let Inst{31-26} = 1;
  let Inst{25-21} = rs;
  let Inst{20-16} = funct;
  let Inst{15-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 414-415
```tablegen
class SYNC_FM : StdArch {
  bits<5> stype;
```
- EN: Declares reusable TableGen class `SYNC_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `SYNC_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 417-417
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 419-422
```tablegen
  let Inst{31-26} = 0;
  let Inst{10-6}  = stype;
  let Inst{5-0}   = 0xf;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 424-428
```tablegen
class SYNCI_FM : StdArch {
  // Produced by the mem_simm16 address as reg << 16 | imm (see getMemEncoding).
  bits<21> addr;
  bits<5> rs = addr{20-16};
  bits<16> offset = addr{15-0};
```
- EN: Declares reusable TableGen class `SYNCI_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `SYNCI_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 430-430
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 432-436
```tablegen
  let Inst{31-26} = 0b000001;
  let Inst{25-21} = rs;
  let Inst{20-16} = 0b11111;
  let Inst{15-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 438-440
```tablegen
class MULT_FM<bits<6> op, bits<6> funct> : StdArch {
  bits<5>  rs;
  bits<5>  rt;
```
- EN: Declares reusable TableGen class `MULT_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `MULT_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 442-442
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 444-449
```tablegen
  let Inst{31-26} = op;
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-6}  = 0;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 451-455
```tablegen
class EXT_FM<bits<6> funct> : StdArch {
  bits<5> rt;
  bits<5> rs;
  bits<5> pos;
  bits<5> size;
```
- EN: Declares reusable TableGen class `EXT_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `EXT_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 457-457
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 459-465
```tablegen
  let Inst{31-26} = 0x1f;
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-11} = size;
  let Inst{10-6}  = pos;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 467-470
```tablegen
class RDHWR_FM : StdArch {
  bits<5> rt;
  bits<5> rd;
  bits<3> sel;
```
- EN: Declares reusable TableGen class `RDHWR_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `RDHWR_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 472-472
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 474-481
```tablegen
  let Inst{31-26} = 0x1f;
  let Inst{25-21} = 0;
  let Inst{20-16} = rt;
  let Inst{15-11} = rd;
  let Inst{10-9}  = 0b00;
  let Inst{8-6}   = sel;
  let Inst{5-0}   = 0x3b;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 483-486
```tablegen
class TEQ_FM<bits<6> funct> : StdArch {
  bits<5> rs;
  bits<5> rt;
  bits<10> code_;
```
- EN: Declares reusable TableGen class `TEQ_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `TEQ_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 488-488
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 490-495
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-6}  = code_;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 497-499
```tablegen
class TEQI_FM<bits<5> funct> : StdArch {
  bits<5> rs;
  bits<16> imm16;
```
- EN: Declares reusable TableGen class `TEQI_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `TEQI_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 501-501
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 503-507
```tablegen
  let Inst{31-26} = 1;
  let Inst{25-21} = rs;
  let Inst{20-16}   = funct;
  let Inst{15-0}  = imm16;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 509-510
```tablegen
class WAIT_FM : StdArch {
  bits<32> Inst;
```
- EN: Declares reusable TableGen class `WAIT_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `WAIT_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 512-516
```tablegen
  let Inst{31-26} = 0x10;
  let Inst{25}    = 1;
  let Inst{24-6}  = 0;
  let Inst{5-0}   = 0x20;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 518-522
```tablegen
class EXTS_FM<bits<6> funct> : StdArch {
  bits<5> rt;
  bits<5> rs;
  bits<5> pos;
  bits<5> lenm1;
```
- EN: Declares reusable TableGen class `EXTS_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `EXTS_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 524-524
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 526-532
```tablegen
  let Inst{31-26} = 0x1c;
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-11} = lenm1;
  let Inst{10-6}  = pos;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 534-535
```tablegen
class MTMR_FM<bits<6> funct> : StdArch {
  bits<5> rs;
```
- EN: Declares reusable TableGen class `MTMR_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `MTMR_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 537-537
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 539-543
```tablegen
  let Inst{31-26} = 0x1c;
  let Inst{25-21} = rs;
  let Inst{20-6}  = 0;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 545-547
```tablegen
class POP_FM<bits<6> funct> : StdArch {
  bits<5> rd;
  bits<5> rs;
```
- EN: Declares reusable TableGen class `POP_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `POP_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 549-549
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 551-557
```tablegen
  let Inst{31-26} = 0x1c;
  let Inst{25-21} = rs;
  let Inst{20-16} = 0;
  let Inst{15-11} = rd;
  let Inst{10-6}  = 0;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 559-562
```tablegen
class SEQ_FM<bits<6> funct> : StdArch {
  bits<5> rd;
  bits<5> rs;
  bits<5> rt;
```
- EN: Declares reusable TableGen class `SEQ_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `SEQ_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 564-564
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 566-572
```tablegen
  let Inst{31-26} = 0x1c;
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-11} = rd;
  let Inst{10-6}  = 0;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 574-577
```tablegen
class SEQI_FM<bits<6> funct> : StdArch {
  bits<5> rs;
  bits<5> rt;
  bits<10> imm10;
```
- EN: Declares reusable TableGen class `SEQI_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `SEQI_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 579-579
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 581-586
```tablegen
  let Inst{31-26} = 0x1c;
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-6}  = imm10;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 588-590
```tablegen
class SAA_FM<bits<6> funct> : StdArch {
  bits<5> rt;
  bits<5> rs;
```
- EN: Declares reusable TableGen class `SAA_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `SAA_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 592-592
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 594-599
```tablegen
  let Inst{31-26} = 0x1c;
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-6}  = 0;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 601-603
```tablegen
//===----------------------------------------------------------------------===//
//  System calls format <op|code_|funct>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 605-612
```tablegen
class SYS_FM<bits<6> funct> : StdArch
{
  bits<20> code_;
  bits<32> Inst;
  let Inst{31-26} = 0x0;
  let Inst{25-6} = code_;
  let Inst{5-0}  = funct;
}
```
- EN: Declares reusable TableGen class `SYS_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `SYS_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 614-616
```tablegen
//===----------------------------------------------------------------------===//
//  Break instruction format <op|code_1|funct>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 618-627
```tablegen
class BRK_FM<bits<6> funct> : StdArch
{
  bits<10> code_1;
  bits<10> code_2;
  bits<32> Inst;
  let Inst{31-26} = 0x0;
  let Inst{25-16} = code_1;
  let Inst{15-6}  = code_2;
  let Inst{5-0}   = funct;
}
```
- EN: Declares reusable TableGen class `BRK_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `BRK_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 629-631
```tablegen
//===----------------------------------------------------------------------===//
//  Exception return format <Cop0|1|0|funct>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 633-641
```tablegen
class ER_FM<bits<6> funct, bit LLBit> : StdArch
{
  bits<32> Inst;
  let Inst{31-26} = 0x10;
  let Inst{25}    = 1;
  let Inst{24-7}  = 0;
  let Inst{6} = LLBit;
  let Inst{5-0}   = funct;
}
```
- EN: Declares reusable TableGen class `ER_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `ER_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 643-645
```tablegen
//===----------------------------------------------------------------------===//
//  Enable/disable interrupt instruction format <Cop0|MFMC0|rt|12|0|sc|0|0>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 647-658
```tablegen
class EI_FM<bits<1> sc> : StdArch
{
  bits<32> Inst;
  bits<5> rt;
  let Inst{31-26} = 0x10;
  let Inst{25-21} = 0xb;
  let Inst{20-16} = rt;
  let Inst{15-11} = 0xc;
  let Inst{10-6}  = 0;
  let Inst{5}     = sc;
  let Inst{4-0}   = 0;
}
```
- EN: Declares reusable TableGen class `EI_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `EI_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 660-671
```tablegen
//===----------------------------------------------------------------------===//
//
//  FLOATING POINT INSTRUCTION FORMATS
//
//  opcode  - operation code.
//  fs      - src reg.
//  ft      - dst reg (on a 2 regs instr) or src reg (on a 3 reg instr).
//  fd      - dst reg, only used on 3 regs instr.
//  fmt     - double or single precision.
//  funct   - combined with opcode field give us an operation code.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 673-675
```tablegen
//===----------------------------------------------------------------------===//
// Format FI instruction class in Mips : <|opcode|base|ft|immediate|>
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 677-680
```tablegen
class ADDS_FM<bits<6> funct, bits<5> fmt> : StdArch {
  bits<5> fd;
  bits<5> fs;
  bits<5> ft;
```
- EN: Declares reusable TableGen class `ADDS_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `ADDS_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 682-682
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 684-690
```tablegen
  let Inst{31-26} = 0x11;
  let Inst{25-21} = fmt;
  let Inst{20-16} = ft;
  let Inst{15-11} = fs;
  let Inst{10-6}  = fd;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 692-694
```tablegen
class ABSS_FM<bits<6> funct, bits<5> fmt> : StdArch {
  bits<5> fd;
  bits<5> fs;
```
- EN: Declares reusable TableGen class `ABSS_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `ABSS_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 696-696
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 698-704
```tablegen
  let Inst{31-26} = 0x11;
  let Inst{25-21} = fmt;
  let Inst{20-16} = 0;
  let Inst{15-11} = fs;
  let Inst{10-6}  = fd;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 706-708
```tablegen
class MFC1_FM<bits<5> funct> : StdArch {
  bits<5> rt;
  bits<5> fs;
```
- EN: Declares reusable TableGen class `MFC1_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `MFC1_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 710-710
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 712-717
```tablegen
  let Inst{31-26} = 0x11;
  let Inst{25-21} = funct;
  let Inst{20-16} = rt;
  let Inst{15-11} = fs;
  let Inst{10-0}  = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 719-721
```tablegen
class LW_FM<bits<6> op> : StdArch {
  bits<5> rt;
  bits<21> addr;
```
- EN: Declares reusable TableGen class `LW_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `LW_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 723-723
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 725-729
```tablegen
  let Inst{31-26} = op;
  let Inst{25-21} = addr{20-16};
  let Inst{20-16} = rt;
  let Inst{15-0}  = addr{15-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 731-735
```tablegen
class MADDS_FM<bits<3> funct, bits<3> fmt> : StdArch {
  bits<5> fd;
  bits<5> fr;
  bits<5> fs;
  bits<5> ft;
```
- EN: Declares reusable TableGen class `MADDS_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `MADDS_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 737-737
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 739-746
```tablegen
  let Inst{31-26} = 0x13;
  let Inst{25-21} = fr;
  let Inst{20-16} = ft;
  let Inst{15-11} = fs;
  let Inst{10-6}  = fd;
  let Inst{5-3}   = funct;
  let Inst{2-0}   = fmt;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 748-751
```tablegen
class LWXC1_FM<bits<6> funct> : StdArch {
  bits<5> fd;
  bits<5> base;
  bits<5> index;
```
- EN: Declares reusable TableGen class `LWXC1_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `LWXC1_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 753-753
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 755-761
```tablegen
  let Inst{31-26} = 0x13;
  let Inst{25-21} = base;
  let Inst{20-16} = index;
  let Inst{15-11} = 0;
  let Inst{10-6}  = fd;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 763-766
```tablegen
class SWXC1_FM<bits<6> funct> : StdArch {
  bits<5> fs;
  bits<5> base;
  bits<5> index;
```
- EN: Declares reusable TableGen class `SWXC1_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `SWXC1_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 768-768
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 770-776
```tablegen
  let Inst{31-26} = 0x13;
  let Inst{25-21} = base;
  let Inst{20-16} = index;
  let Inst{15-11} = fs;
  let Inst{10-6}  = 0;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 778-780
```tablegen
class BC1F_FM<bit nd, bit tf> : StdArch {
  bits<3>  fcc;
  bits<16> offset;
```
- EN: Declares reusable TableGen class `BC1F_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `BC1F_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 782-782
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 784-790
```tablegen
  let Inst{31-26} = 0x11;
  let Inst{25-21} = 0x8;
  let Inst{20-18} = fcc;
  let Inst{17} = nd;
  let Inst{16} = tf;
  let Inst{15-0} = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 792-796
```tablegen
class CEQS_FM<bits<5> fmt> : StdArch {
  bits<5> fs;
  bits<5> ft;
  bits<3> fcc;
  bits<4> cond;
```
- EN: Declares reusable TableGen class `CEQS_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `CEQS_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 798-798
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 800-807
```tablegen
  let Inst{31-26} = 0x11;
  let Inst{25-21} = fmt;
  let Inst{20-16} = ft;
  let Inst{15-11} = fs;
  let Inst{10-8} = fcc;
  let Inst{7-4} = 0x3;
  let Inst{3-0} = cond;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 809-811
```tablegen
class C_COND_FM<bits<5> fmt, bits<4> c> : CEQS_FM<fmt> {
  let cond = c;
}
```
- EN: Declares reusable TableGen class `C_COND_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `C_COND_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 813-816
```tablegen
class CMov_I_F_FM<bits<6> funct, bits<5> fmt> : StdArch {
  bits<5> fd;
  bits<5> fs;
  bits<5> rt;
```
- EN: Declares reusable TableGen class `CMov_I_F_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `CMov_I_F_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 818-818
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 820-826
```tablegen
  let Inst{31-26} = 0x11;
  let Inst{25-21} = fmt;
  let Inst{20-16} = rt;
  let Inst{15-11} = fs;
  let Inst{10-6} = fd;
  let Inst{5-0} = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 828-831
```tablegen
class CMov_F_I_FM<bit tf> : StdArch {
  bits<5> rd;
  bits<5> rs;
  bits<3> fcc;
```
- EN: Declares reusable TableGen class `CMov_F_I_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `CMov_F_I_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 833-833
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 835-843
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-21} = rs;
  let Inst{20-18} = fcc;
  let Inst{17} = 0;
  let Inst{16} = tf;
  let Inst{15-11} = rd;
  let Inst{10-6} = 0;
  let Inst{5-0} = 1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 845-848
```tablegen
class CMov_F_F_FM<bits<5> fmt, bit tf> : StdArch {
  bits<5> fd;
  bits<5> fs;
  bits<3> fcc;
```
- EN: Declares reusable TableGen class `CMov_F_F_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `CMov_F_F_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 850-850
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 852-860
```tablegen
  let Inst{31-26} = 0x11;
  let Inst{25-21} = fmt;
  let Inst{20-18} = fcc;
  let Inst{17} = 0;
  let Inst{16} = tf;
  let Inst{15-11} = fs;
  let Inst{10-6} = fd;
  let Inst{5-0} = 0x11;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 862-863
```tablegen
class BARRIER_FM<bits<5> op> : StdArch {
  bits<32> Inst;
```
- EN: Declares reusable TableGen class `BARRIER_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `BARRIER_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 865-871
```tablegen
  let Inst{31-26} = 0; // SPECIAL
  let Inst{25-21} = 0;
  let Inst{20-16} = 0; // rt = 0
  let Inst{15-11} = 0; // rd = 0
  let Inst{10-6} = op; // Operation
  let Inst{5-0} = 0;   // SLL
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 873-874
```tablegen
class SDBBP_FM : StdArch {
  bits<20> code_;
```
- EN: Declares reusable TableGen class `SDBBP_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `SDBBP_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 876-876
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 878-881
```tablegen
  let Inst{31-26} = 0b011100; // SPECIAL2
  let Inst{25-6} = code_;
  let Inst{5-0} = 0b111111;   // SDBBP
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 883-884
```tablegen
class JR_HB_FM<bits<6> op> : StdArch{
  bits<5> rs;
```
- EN: Declares reusable TableGen class `JR_HB_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `JR_HB_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 886-886
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 888-894
```tablegen
  let Inst{31-26} = 0; // SPECIAL
  let Inst{25-21} = rs;
  let Inst{20-11} = 0;
  let Inst{10} = 1;
  let Inst{9-6} = 0;
  let Inst{5-0} = op;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 896-898
```tablegen
class JALR_HB_FM<bits<6> op> : StdArch {
  bits<5> rd;
  bits<5> rs;
```
- EN: Declares reusable TableGen class `JALR_HB_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `JALR_HB_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 900-900
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 902-909
```tablegen
  let Inst{31-26} = 0; // SPECIAL
  let Inst{25-21} = rs;
  let Inst{20-16} = 0;
  let Inst{15-11} = rd;
  let Inst{10} = 1;
  let Inst{9-6} = 0;
  let Inst{5-0} = op;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 911-912
```tablegen
class COP0_TLB_FM<bits<6> op> : StdArch {
  bits<32> Inst;
```
- EN: Declares reusable TableGen class `COP0_TLB_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `COP0_TLB_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 914-918
```tablegen
  let Inst{31-26} = 0x10; // COP0
  let Inst{25} = 1;       // CO
  let Inst{24-6} = 0;
  let Inst{5-0} = op;     // Operation
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 920-924
```tablegen
class CACHEOP_FM<bits<6> op> : StdArch {
  bits<21> addr;
  bits<5> hint;
  bits<5> base = addr{20-16};
  bits<16> offset = addr{15-0};
```
- EN: Declares reusable TableGen class `CACHEOP_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `CACHEOP_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 926-926
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 928-932
```tablegen
  let Inst{31-26} = op;
  let Inst{25-21} = base;
  let Inst{20-16} = hint;
  let Inst{15-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 934-935
```tablegen
class HYPCALL_FM<bits<6> op> : StdArch {
  bits<10> code_;
```
- EN: Declares reusable TableGen class `HYPCALL_FM` for `MipsInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFormats` 声明可复用的 TableGen 类 `HYPCALL_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 937-937
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 939-943
```tablegen
  let Inst{31-26} = 0b010000;
  let Inst{25}    = 1;
  let Inst{20-11} = code_;
  let Inst{5-0}   = op;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

## Key Concepts / 关键概念

- EN: Primary role: instruction encoding formats and bitfield layout.
  - CN: 核心职责：指令编码格式与位域布局。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。
- EN: SelectionDAG patterns or node profiles connect IR-level intent to target instructions.
  - CN: SelectionDAG 模式或节点轮廓把 IR 层意图连接到目标指令。
- EN: Bitfield assignments describe exact instruction encodings and per-instruction flags.
  - CN: 位字段赋值用于描述精确的指令编码以及每条指令的标志位。
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `(none)`.
  - CN: TableGen 包含项：`(none)`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
