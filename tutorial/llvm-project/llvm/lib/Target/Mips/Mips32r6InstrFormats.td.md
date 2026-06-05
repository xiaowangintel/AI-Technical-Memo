# Mips32r6InstrFormats.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/Mips32r6InstrFormats.td`
- Repository: `llvm-project`
- Purpose (EN): This file describes Mips32r6 instruction formats.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `Mips32r6InstrFormats`，涵盖指令编码格式与位域布局。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//=- Mips32r6InstrFormats.td - Mips32r6 Instruction Formats -*- tablegen -*-==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes Mips32r6 instruction formats.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 13-13
```tablegen
class R6MMR6Rel;
```
- EN: Declares reusable TableGen class `R6MMR6Rel` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `R6MMR6Rel`，通常用于抽象共享字段、谓词或编码结构。

### Lines 15-25
```tablegen
def MipsR62MicroMipsR6 : InstrMapping {
  let FilterClass = "R6MMR6Rel";
  // Instructions with the same BaseOpcode and isNVStore values form a row.
  let RowFields = ["BaseOpcode"];
  // Instructions with the same predicate sense form a column.
  let ColFields = ["Arch"];
  // The key column is the unpredicated instructions.
  let KeyCol = ["mipsr6"];
  // Value columns are PredSense=true and PredSense=false
  let ValueCols = [["mipsr6"], ["micromipsr6"]];
}
```
- EN: Defines TableGen record `MipsR62MicroMipsR6` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MipsR62MicroMipsR6`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 27-30
```tablegen
class MipsR6Arch<string opstr> {
  string Arch = "mipsr6";
  string BaseOpcode = opstr;
}
```
- EN: Declares reusable TableGen class `MipsR6Arch` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `MipsR6Arch`，通常用于抽象共享字段、谓词或编码结构。

### Lines 32-35
```tablegen
class MipsR6Inst : MipsInst<(outs), (ins), "", [], FrmOther> {
  let DecoderNamespace = "Mips32r6_64r6";
  let EncodingPredicates = [HasStdEnc];
}
```
- EN: Declares reusable TableGen class `MipsR6Inst` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `MipsR6Inst`，通常用于抽象共享字段、谓词或编码结构。

### Lines 37-41
```tablegen
//===----------------------------------------------------------------------===//
//
// Field Values
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 43-60
```tablegen
class OPGROUP<bits<6> Val> {
  bits<6> Value = Val;
}
def OPGROUP_COP0     : OPGROUP<0b010000>;
def OPGROUP_COP1     : OPGROUP<0b010001>;
def OPGROUP_COP2     : OPGROUP<0b010010>;
def OPGROUP_ADDI     : OPGROUP<0b001000>;
def OPGROUP_AUI      : OPGROUP<0b001111>;
def OPGROUP_BLEZ     : OPGROUP<0b000110>;
def OPGROUP_BGTZ     : OPGROUP<0b000111>;
def OPGROUP_BLEZL    : OPGROUP<0b010110>;
def OPGROUP_BGTZL    : OPGROUP<0b010111>;
def OPGROUP_DADDI    : OPGROUP<0b011000>;
def OPGROUP_DAUI     : OPGROUP<0b011101>;
def OPGROUP_PCREL    : OPGROUP<0b111011>;
def OPGROUP_REGIMM   : OPGROUP<0b000001>;
def OPGROUP_SPECIAL  : OPGROUP<0b000000>;
// The spec occasionally names this value LL, LLD, SC, or SCD.
```
- EN: Declares reusable TableGen class `OPGROUP` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `OPGROUP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 61-63
```tablegen
def OPGROUP_SPECIAL3 : OPGROUP<0b011111>;
// The spec names this constant LWC2, LDC2, SWC2, and SDC2 in different places.
def OPGROUP_COP2LDST : OPGROUP<0b010010>;
```
- EN: Defines TableGen record `OPGROUP_SPECIAL3` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `OPGROUP_SPECIAL3`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 65-70
```tablegen
class OPCODE2<bits<2> Val> {
  bits<2> Value = Val;
}
def OPCODE2_ADDIUPC : OPCODE2<0b00>;
def OPCODE2_LWPC    : OPCODE2<0b01>;
def OPCODE2_LWUPC   : OPCODE2<0b10>;
```
- EN: Declares reusable TableGen class `OPCODE2` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `OPCODE2`，通常用于抽象共享字段、谓词或编码结构。

### Lines 72-75
```tablegen
class OPCODE3<bits<3> Val> {
  bits<3> Value = Val;
}
def OPCODE3_LDPC : OPCODE3<0b110>;
```
- EN: Declares reusable TableGen class `OPCODE3` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `OPCODE3`，通常用于抽象共享字段、谓词或编码结构。

### Lines 77-94
```tablegen
class OPCODE5<bits<5> Val> {
  bits<5> Value = Val;
}
def OPCODE5_ALUIPC : OPCODE5<0b11111>;
def OPCODE5_AUIPC  : OPCODE5<0b11110>;
def OPCODE5_DAHI : OPCODE5<0b00110>;
def OPCODE5_DATI : OPCODE5<0b11110>;
def OPCODE5_BC1EQZ : OPCODE5<0b01001>;
def OPCODE5_BC1NEZ : OPCODE5<0b01101>;
def OPCODE5_BC2EQZ : OPCODE5<0b01001>;
def OPCODE5_BC2NEZ : OPCODE5<0b01101>;
def OPCODE5_BGEZAL : OPCODE5<0b10001>;
def OPCODE5_NAL : OPCODE5<0b10000>;
def OPCODE5_SIGRIE : OPCODE5<0b10111>;
// The next four constants are unnamed in the spec. These names are taken from
// the OPGROUP names they are used with.
def OPCODE5_LDC2   : OPCODE5<0b01110>;
def OPCODE5_LWC2   : OPCODE5<0b01010>;
```
- EN: Declares reusable TableGen class `OPCODE5` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `OPCODE5`，通常用于抽象共享字段、谓词或编码结构。

### Lines 95-96
```tablegen
def OPCODE5_SDC2   : OPCODE5<0b01111>;
def OPCODE5_SWC2   : OPCODE5<0b01011>;
```
- EN: Defines TableGen record `OPCODE5_SDC2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `OPCODE5_SDC2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 98-115
```tablegen
class OPCODE6<bits<6> Val> {
  bits<6> Value = Val;
}
def OPCODE6_ALIGN    : OPCODE6<0b100000>;
def OPCODE6_DALIGN   : OPCODE6<0b100100>;
def OPCODE6_BITSWAP  : OPCODE6<0b100000>;
def OPCODE6_DBITSWAP : OPCODE6<0b100100>;
def OPCODE6_JALR     : OPCODE6<0b001001>;
def OPCODE6_CACHE    : OPCODE6<0b100101>;
def OPCODE6_PREF     : OPCODE6<0b110101>;
// The next four constants are unnamed in the spec. These names are taken from
// the OPGROUP names they are used with.
def OPCODE6_LL       : OPCODE6<0b110110>;
def OPCODE6_LLD      : OPCODE6<0b110111>;
def OPCODE6_SC       : OPCODE6<0b100110>;
def OPCODE6_SCD      : OPCODE6<0b100111>;
def OPCODE6_CLO      : OPCODE6<0b010001>;
def OPCODE6_CLZ      : OPCODE6<0b010000>;
```
- EN: Declares reusable TableGen class `OPCODE6` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `OPCODE6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 116-120
```tablegen
def OPCODE6_DCLO     : OPCODE6<0b010011>;
def OPCODE6_DCLZ     : OPCODE6<0b010010>;
def OPCODE6_LSA      : OPCODE6<0b000101>;
def OPCODE6_DLSA     : OPCODE6<0b010101>;
def OPCODE6_SDBBP    : OPCODE6<0b001110>;
```
- EN: Defines TableGen record `OPCODE6_DCLO` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `OPCODE6_DCLO`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 122-126
```tablegen
class FIELD_FMT<bits<5> Val> {
  bits<5> Value = Val;
}
def FIELD_FMT_S : FIELD_FMT<0b10000>;
def FIELD_FMT_D : FIELD_FMT<0b10001>;
```
- EN: Declares reusable TableGen class `FIELD_FMT` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `FIELD_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 128-145
```tablegen
class FIELD_CMP_COND<bits<5> Val> {
  bits<5> Value = Val;
}
// Note: The CMP_COND_FMT names differ from the C_COND_FMT names.
def FIELD_CMP_COND_AF   : FIELD_CMP_COND<0b00000>;
def FIELD_CMP_COND_UN   : FIELD_CMP_COND<0b00001>;
def FIELD_CMP_COND_EQ   : FIELD_CMP_COND<0b00010>;
def FIELD_CMP_COND_UEQ  : FIELD_CMP_COND<0b00011>;
def FIELD_CMP_COND_LT   : FIELD_CMP_COND<0b00100>;
def FIELD_CMP_COND_ULT  : FIELD_CMP_COND<0b00101>;
def FIELD_CMP_COND_LE   : FIELD_CMP_COND<0b00110>;
def FIELD_CMP_COND_ULE  : FIELD_CMP_COND<0b00111>;
def FIELD_CMP_COND_SAF  : FIELD_CMP_COND<0b01000>;
def FIELD_CMP_COND_SUN  : FIELD_CMP_COND<0b01001>;
def FIELD_CMP_COND_SEQ  : FIELD_CMP_COND<0b01010>;
def FIELD_CMP_COND_SUEQ : FIELD_CMP_COND<0b01011>;
def FIELD_CMP_COND_SLT  : FIELD_CMP_COND<0b01100>;
def FIELD_CMP_COND_SULT : FIELD_CMP_COND<0b01101>;
```
- EN: Declares reusable TableGen class `FIELD_CMP_COND` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `FIELD_CMP_COND`，通常用于抽象共享字段、谓词或编码结构。

### Lines 146-147
```tablegen
def FIELD_CMP_COND_SLE  : FIELD_CMP_COND<0b01110>;
def FIELD_CMP_COND_SULE : FIELD_CMP_COND<0b01111>;
```
- EN: Defines TableGen record `FIELD_CMP_COND_SLE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FIELD_CMP_COND_SLE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 149-153
```tablegen
class FIELD_CMP_FORMAT<bits<5> Val> {
  bits<5> Value = Val;
}
def FIELD_CMP_FORMAT_S : FIELD_CMP_FORMAT<0b10100>;
def FIELD_CMP_FORMAT_D : FIELD_CMP_FORMAT<0b10101>;
```
- EN: Declares reusable TableGen class `FIELD_CMP_FORMAT` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `FIELD_CMP_FORMAT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 155-161
```tablegen
//===----------------------------------------------------------------------===//
//
// Disambiguators
//
//===----------------------------------------------------------------------===//
//
// Some encodings are ambiguous except by comparing field values.
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 163-165
```tablegen
class DecodeDisambiguates<string Name> {
  string DecoderMethod = !strconcat("Decode", Name);
}
```
- EN: Declares reusable TableGen class `DecodeDisambiguates` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `DecodeDisambiguates`，通常用于抽象共享字段、谓词或编码结构。

### Lines 167-169
```tablegen
class DecodeDisambiguatedBy<string Name> : DecodeDisambiguates<Name> {
  string DecoderNamespace = "Mips32r6_64r6_Ambiguous";
}
```
- EN: Declares reusable TableGen class `DecodeDisambiguatedBy` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `DecodeDisambiguatedBy`，通常用于抽象共享字段、谓词或编码结构。

### Lines 171-175
```tablegen
//===----------------------------------------------------------------------===//
//
// Encoding Formats
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 177-180
```tablegen
class AUI_FM : MipsR6Inst {
  bits<5> rs;
  bits<5> rt;
  bits<16> imm;
```
- EN: Declares reusable TableGen class `AUI_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `AUI_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 182-182
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 184-188
```tablegen
  let Inst{31-26} = OPGROUP_AUI.Value;
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-0} = imm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 190-192
```tablegen
class DAUI_FM : AUI_FM {
  let Inst{31-26} = OPGROUP_DAUI.Value;
}
```
- EN: Declares reusable TableGen class `DAUI_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `DAUI_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 194-195
```tablegen
class BAL_FM : MipsR6Inst {
  bits<16> offset;
```
- EN: Declares reusable TableGen class `BAL_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `BAL_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 197-197
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 199-203
```tablegen
  let Inst{31-26} = OPGROUP_REGIMM.Value;
  let Inst{25-21} = 0b00000;
  let Inst{20-16} = OPCODE5_BGEZAL.Value;
  let Inst{15-0} = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 205-207
```tablegen
// NAL for Release 6
class NAL_FM : MipsR6Inst {
  bits<32> Inst;
```
- EN: Declares reusable TableGen class `NAL_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `NAL_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 209-213
```tablegen
  let Inst{31-26} = OPGROUP_REGIMM.Value;
  let Inst{25-21} = 0b00000; 
  let Inst{20-16} = OPCODE5_NAL.Value;
  let Inst{15-0} = 0x00;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 215-216
```tablegen
class COP0_EVP_DVP_FM<bits<1> sc> : MipsR6Inst {
  bits<5> rt;
```
- EN: Declares reusable TableGen class `COP0_EVP_DVP_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `COP0_EVP_DVP_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 218-218
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 220-228
```tablegen
  let Inst{31-26} = OPGROUP_COP0.Value;
  let Inst{25-21} = 0b01011;
  let Inst{20-16} = rt;
  let Inst{15-11} = 0b00000;
  let Inst{10-6}  = 0b00000;
  let Inst{5}     = sc;
  let Inst{4-3}   = 0b00;
  let Inst{2-0}   = 0b100;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 230-232
```tablegen
class COP1_2R_FM<bits<6> funct, FIELD_FMT Format> : MipsR6Inst {
  bits<5> fs;
  bits<5> fd;
```
- EN: Declares reusable TableGen class `COP1_2R_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `COP1_2R_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 234-234
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 236-242
```tablegen
  let Inst{31-26} = OPGROUP_COP1.Value;
  let Inst{25-21} = Format.Value;
  let Inst{20-16} = 0b00000;
  let Inst{15-11} = fs;
  let Inst{10-6}  = fd;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 244-247
```tablegen
class COP1_3R_FM<bits<6> funct, FIELD_FMT Format> : MipsR6Inst {
  bits<5> ft;
  bits<5> fs;
  bits<5> fd;
```
- EN: Declares reusable TableGen class `COP1_3R_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `COP1_3R_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 249-249
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 251-257
```tablegen
  let Inst{31-26} = OPGROUP_COP1.Value;
  let Inst{25-21} = Format.Value;
  let Inst{20-16} = ft;
  let Inst{15-11} = fs;
  let Inst{10-6} = fd;
  let Inst{5-0} = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 259-261
```tablegen
class COP1_BCCZ_FM<OPCODE5 Operation> : MipsR6Inst {
  bits<5> ft;
  bits<16> offset;
```
- EN: Declares reusable TableGen class `COP1_BCCZ_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `COP1_BCCZ_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 263-263
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 265-269
```tablegen
  let Inst{31-26} = OPGROUP_COP1.Value;
  let Inst{25-21} = Operation.Value;
  let Inst{20-16} = ft;
  let Inst{15-0} = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 271-273
```tablegen
class COP2_BCCZ_FM<OPCODE5 Operation> : MipsR6Inst {
  bits<5> ct;
  bits<16> offset;
```
- EN: Declares reusable TableGen class `COP2_BCCZ_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `COP2_BCCZ_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 275-275
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 277-281
```tablegen
  let Inst{31-26} = OPGROUP_COP2.Value;
  let Inst{25-21} = Operation.Value;
  let Inst{20-16} = ct;
  let Inst{15-0} = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 283-285
```tablegen
class PCREL16_FM<OPCODE5 Operation> : MipsR6Inst {
  bits<5> rs;
  bits<16> imm;
```
- EN: Declares reusable TableGen class `PCREL16_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `PCREL16_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 287-287
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 289-293
```tablegen
  let Inst{31-26} = OPGROUP_PCREL.Value;
  let Inst{25-21} = rs;
  let Inst{20-16} = Operation.Value;
  let Inst{15-0} = imm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 295-297
```tablegen
class PCREL19_FM<OPCODE2 Operation> : MipsR6Inst {
  bits<5> rs;
  bits<19> imm;
```
- EN: Declares reusable TableGen class `PCREL19_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `PCREL19_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 299-299
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 301-305
```tablegen
  let Inst{31-26} = OPGROUP_PCREL.Value;
  let Inst{25-21} = rs;
  let Inst{20-19} = Operation.Value;
  let Inst{18-0} = imm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 307-309
```tablegen
class PCREL18_FM<OPCODE3 Operation> : MipsR6Inst {
  bits<5> rs;
  bits<18> imm;
```
- EN: Declares reusable TableGen class `PCREL18_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `PCREL18_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 311-311
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 313-317
```tablegen
  let Inst{31-26} = OPGROUP_PCREL.Value;
  let Inst{25-21} = rs;
  let Inst{20-18} = Operation.Value;
  let Inst{17-0} = imm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 319-321
```tablegen
class SPECIAL3_2R_FM<OPCODE6 Operation> : MipsR6Inst {
  bits<5> rd;
  bits<5> rt;
```
- EN: Declares reusable TableGen class `SPECIAL3_2R_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `SPECIAL3_2R_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 323-323
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 325-331
```tablegen
  let Inst{31-26} = OPGROUP_SPECIAL3.Value;
  let Inst{25-21} = 0b00000;
  let Inst{20-16} = rt;
  let Inst{15-11} = rd;
  let Inst{10-6}  = 0b00000;
  let Inst{5-0}   = Operation.Value;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 333-337
```tablegen
class SPECIAL3_MEM_FM<OPCODE6 Operation> : MipsR6Inst {
  bits<21> addr;
  bits<5> hint;
  bits<5> base = addr{20-16};
  bits<9> offset = addr{8-0};
```
- EN: Declares reusable TableGen class `SPECIAL3_MEM_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `SPECIAL3_MEM_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 339-339
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 341-347
```tablegen
  let Inst{31-26} = OPGROUP_SPECIAL3.Value;
  let Inst{25-21} = base;
  let Inst{20-16} = hint;
  let Inst{15-7}  = offset;
  let Inst{6}     = 0;
  let Inst{5-0}   = Operation.Value;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 349-351
```tablegen
class SPECIAL_2R_FM<OPCODE6 Operation> : MipsR6Inst {
  bits<5> rd;
  bits<5> rs;
```
- EN: Declares reusable TableGen class `SPECIAL_2R_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `SPECIAL_2R_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 353-353
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 355-361
```tablegen
  let Inst{31-26} = OPGROUP_SPECIAL.Value;
  let Inst{25-21} = rs;
  let Inst{20-16} = 0b00000;
  let Inst{15-11} = rd;
  let Inst{10-6}  = 0b00001;
  let Inst{5-0}   = Operation.Value;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 363-366
```tablegen
class SPECIAL_3R_FM<bits<5> mulop, bits<6> funct> : MipsR6Inst {
  bits<5> rd;
  bits<5> rs;
  bits<5> rt;
```
- EN: Declares reusable TableGen class `SPECIAL_3R_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `SPECIAL_3R_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 368-368
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 370-376
```tablegen
  let Inst{31-26} = OPGROUP_SPECIAL.Value;
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-11} = rd;
  let Inst{10-6}  = mulop;
  let Inst{5-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 378-379
```tablegen
class SPECIAL_SDBBP_FM : MipsR6Inst {
  bits<20> code_;
```
- EN: Declares reusable TableGen class `SPECIAL_SDBBP_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `SPECIAL_SDBBP_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 381-381
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 383-386
```tablegen
  let Inst{31-26} = OPGROUP_SPECIAL.Value;
  let Inst{25-6}  = code_;
  let Inst{5-0}   = OPCODE6_SDBBP.Value;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 388-393
```tablegen
// This class is ambiguous with other branches:
//   BEQC/BNEC require that rs < rt && rs != 0
class CMP_BRANCH_2R_OFF16_FM<OPGROUP funct> : MipsR6Inst {
  bits<5> rs;
  bits<5> rt;
  bits<16> offset;
```
- EN: Declares reusable TableGen class `is` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `is`，通常用于抽象共享字段、谓词或编码结构。

### Lines 395-395
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 397-401
```tablegen
  let Inst{31-26} = funct.Value;
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-0} = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 403-408
```tablegen
// This class is ambiguous with other branches:
//   BLEZC/BGEZC/BEQZALC/BNEZALC/BGTZALC require that rs == 0 && rt != 0
// The '1R_RT' in the name means 1 register in the rt field.
class CMP_BRANCH_1R_RT_OFF16_FM<OPGROUP funct> : MipsR6Inst {
  bits<5> rt;
  bits<16> offset;
```
- EN: Declares reusable TableGen class `is` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `is`，通常用于抽象共享字段、谓词或编码结构。

### Lines 410-410
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 412-416
```tablegen
  let Inst{31-26} = funct.Value;
  let Inst{25-21} = 0b00000;
  let Inst{20-16} = rt;
  let Inst{15-0} = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 418-423
```tablegen
// This class is ambiguous with other branches:
//   BLTZC/BGTZC/BLTZALC/BGEZALC require that rs == rt && rt != 0
// The '1R_BOTH' in the name means 1 register in both the rs and rt fields.
class CMP_BRANCH_1R_BOTH_OFF16_FM<OPGROUP funct> : MipsR6Inst {
  bits<5> rt;
  bits<16> offset;
```
- EN: Declares reusable TableGen class `is` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `is`，通常用于抽象共享字段、谓词或编码结构。

### Lines 425-425
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 427-431
```tablegen
  let Inst{31-26} = funct.Value;
  let Inst{25-21} = rt;
  let Inst{20-16} = rt;
  let Inst{15-0} = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 433-435
```tablegen
class CMP_BRANCH_OFF21_FM<bits<6> funct> : MipsR6Inst {
  bits<5> rs; // rs != 0
  bits<21> offset;
```
- EN: Declares reusable TableGen class `CMP_BRANCH_OFF21_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `CMP_BRANCH_OFF21_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 437-437
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 439-442
```tablegen
  let Inst{31-26} = funct;
  let Inst{25-21} = rs;
  let Inst{20-0} = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 444-446
```tablegen
class JMP_IDX_COMPACT_FM<bits<6> funct> : MipsR6Inst {
  bits<5> rt;
  bits<16> offset;
```
- EN: Declares reusable TableGen class `JMP_IDX_COMPACT_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `JMP_IDX_COMPACT_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 448-448
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 450-454
```tablegen
  let Inst{31-26} = funct;
  let Inst{25-21} = 0b00000;
  let Inst{20-16} = rt;
  let Inst{15-0} = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 456-458
```tablegen
class BRANCH_OFF26_FM<bits<6> funct> : MipsR6Inst {
  bits<32> Inst;
  bits<26> offset;
```
- EN: Declares reusable TableGen class `BRANCH_OFF26_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `BRANCH_OFF26_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 460-462
```tablegen
  let Inst{31-26} = funct;
  let Inst{25-0} = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 464-468
```tablegen
class SPECIAL3_ALIGN_FM<OPCODE6 Operation> : MipsR6Inst {
  bits<5> rd;
  bits<5> rs;
  bits<5> rt;
  bits<2> bp;
```
- EN: Declares reusable TableGen class `SPECIAL3_ALIGN_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `SPECIAL3_ALIGN_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 470-470
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 472-479
```tablegen
  let Inst{31-26} = OPGROUP_SPECIAL3.Value;
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-11} = rd;
  let Inst{10-8}  = 0b010;
  let Inst{7-6}   = bp;
  let Inst{5-0}   = Operation.Value;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 481-485
```tablegen
class SPECIAL3_DALIGN_FM<OPCODE6 Operation> : MipsR6Inst {
  bits<5> rd;
  bits<5> rs;
  bits<5> rt;
  bits<3> bp;
```
- EN: Declares reusable TableGen class `SPECIAL3_DALIGN_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `SPECIAL3_DALIGN_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 487-487
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 489-496
```tablegen
  let Inst{31-26} = OPGROUP_SPECIAL3.Value;
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-11} = rd;
  let Inst{10-9}  = 0b01;
  let Inst{8-6}   = bp;
  let Inst{5-0}   = Operation.Value;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 498-502
```tablegen
class SPECIAL3_LL_SC_FM<OPCODE6 Operation> : MipsR6Inst {
  bits<5> rt;
  bits<21> addr;
  bits<5> base = addr{20-16};
  bits<9> offset = addr{8-0};
```
- EN: Declares reusable TableGen class `SPECIAL3_LL_SC_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `SPECIAL3_LL_SC_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 504-504
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 506-510
```tablegen
  let Inst{31-26} = OPGROUP_SPECIAL3.Value;
  let Inst{25-21} = base;
  let Inst{20-16} = rt;
  let Inst{15-7} = offset;
  let Inst{5-0} = Operation.Value;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 512-513
```tablegen
  string DecoderMethod = "DecodeSpecial3LlSc";
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 515-519
```tablegen
class SPECIAL_LSA_FM<OPCODE6 Operation> : MipsR6Inst {
  bits<5> rd;
  bits<5> rs;
  bits<5> rt;
  bits<2> imm2;
```
- EN: Declares reusable TableGen class `SPECIAL_LSA_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `SPECIAL_LSA_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 521-521
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 523-530
```tablegen
  let Inst{31-26} = OPGROUP_SPECIAL.Value;
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-11} = rd;
  let Inst{10-8}  = 0b000;
  let Inst{7-6}   = imm2;
  let Inst{5-0}   = Operation.Value;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 532-534
```tablegen
class REGIMM_FM<OPCODE5 Operation> : MipsR6Inst {
  bits<5> rs;
  bits<16> imm;
```
- EN: Declares reusable TableGen class `REGIMM_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `REGIMM_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 536-536
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 538-542
```tablegen
  let Inst{31-26} = OPGROUP_REGIMM.Value;
  let Inst{25-21} = rs;
  let Inst{20-16} = Operation.Value;
  let Inst{15-0} = imm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 544-548
```tablegen
class COP1_CMP_CONDN_FM<FIELD_CMP_FORMAT Format,
                        FIELD_CMP_COND Cond> : MipsR6Inst {
  bits<5> fd;
  bits<5> fs;
  bits<5> ft;
```
- EN: Declares reusable TableGen class `COP1_CMP_CONDN_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `COP1_CMP_CONDN_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 550-550
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 552-559
```tablegen
  let Inst{31-26} = OPGROUP_COP1.Value;
  let Inst{25-21} = Format.Value;
  let Inst{20-16} = ft;
  let Inst{15-11} = fs;
  let Inst{10-6}  = fd;
  let Inst{5}     = 0;
  let Inst{4-0}   = Cond.Value;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 561-562
```tablegen
class JR_HB_R6_FM<OPCODE6 Operation> : MipsR6Inst {
  bits<5> rs;
```
- EN: Declares reusable TableGen class `JR_HB_R6_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `JR_HB_R6_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 564-564
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 566-573
```tablegen
  let Inst{31-26} = OPGROUP_SPECIAL.Value;
  let Inst{25-21} = rs;
  let Inst{20-16} = 0;
  let Inst{15-11} = 0;
  let Inst{10} = 1;
  let Inst{9-6} = 0;
  let Inst{5-0} = Operation.Value;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 575-579
```tablegen
class COP2LDST_FM<OPCODE5 Operation> : MipsR6Inst {
  bits<5> rt;
  bits<21> addr;
  bits<5> base = addr{20-16};
  bits<11> offset = addr{10-0};
```
- EN: Declares reusable TableGen class `COP2LDST_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `COP2LDST_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 581-581
```tablegen
  bits<32> Inst;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 583-588
```tablegen
  let Inst{31-26} = OPGROUP_COP2LDST.Value;
  let Inst{25-21} = Operation.Value;
  let Inst{20-16} = rt;
  let Inst{15-11} = base;
  let Inst{10-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 590-592
```tablegen
class SPECIAL3_2R_SZ_CRC<bits<2> sz, bits<3> direction> : MipsR6Inst {
  bits<5> rs;
  bits<5> rt;
```
- EN: Declares reusable TableGen class `SPECIAL3_2R_SZ_CRC` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `SPECIAL3_2R_SZ_CRC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 594-600
```tablegen
  let Inst{31-26} = OPGROUP_SPECIAL3.Value;
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-11} = 0b00000;
  let Inst{10-8} = direction;
  let Inst{7-6} = sz;
  let Inst{5-0} = 0b001111;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 602-603
```tablegen
  string DecoderMethod = "DecodeCRC";
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 605-607
```tablegen
class SPECIAL3_GINV<bits<2> ginv> : MipsR6Inst {
  bits<5> rs;
  bits<2> type_;
```
- EN: Declares reusable TableGen class `SPECIAL3_GINV` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `SPECIAL3_GINV`，通常用于抽象共享字段、谓词或编码结构。

### Lines 609-615
```tablegen
  let Inst{31-26} = OPGROUP_SPECIAL3.Value;
  let Inst{25-21} = rs;
  let Inst{20-10} = 0x0;
  let Inst{9-8}   = type_;
  let Inst{7-6}   = ginv;
  let Inst{5-0}   = 0b111101;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 617-618
```tablegen
class SIGRIE_FM : MipsR6Inst {
  bits<16> code_;
```
- EN: Declares reusable TableGen class `SIGRIE_FM` for `Mips32r6InstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips32r6InstrFormats` 声明可复用的 TableGen 类 `SIGRIE_FM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 620-624
```tablegen
  let Inst{31-26} = OPGROUP_REGIMM.Value;
  let Inst{25-21} = 0;
  let Inst{20-16} = OPCODE5_SIGRIE.Value;
  let Inst{15-0} = code_;
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
