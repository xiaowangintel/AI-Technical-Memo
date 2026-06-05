# MipsMSAInstrFormats.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsMSAInstrFormats.td`
- Repository: `llvm-project`
- Purpose (EN): Defines `MipsMSAInstrFormats` in LLVM TableGen DSL for the Mips backend, covering instruction encoding formats and bitfield layout.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `MipsMSAInstrFormats`，涵盖指令编码格式与位域布局。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===- MipsMSAInstrFormats.td - Mips Instruction Formats ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 9-13
```tablegen
class MSAInst : MipsInst<(outs), (ins), "", [], FrmOther>,
                ASE_MSA {
  let EncodingPredicates = [HasStdEnc];
  let Inst{31-26} = 0b011110;
}
```
- EN: Declares reusable TableGen class `MSAInst` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSAInst`，通常用于抽象共享字段、谓词或编码结构。

### Lines 15-17
```tablegen
class MSACBranch : MSAInst {
  let Inst{31-26} = 0b010001;
}
```
- EN: Declares reusable TableGen class `MSACBranch` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSACBranch`，通常用于抽象共享字段、谓词或编码结构。

### Lines 19-21
```tablegen
class MSASpecial : MSAInst {
  let Inst{31-26} = 0b000000;
}
```
- EN: Declares reusable TableGen class `MSASpecial` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSASpecial`，通常用于抽象共享字段、谓词或编码结构。

### Lines 23-27
```tablegen
class MSAPseudo<dag outs, dag ins, list<dag> pattern>:
  MipsPseudo<outs, ins, pattern> {
  let EncodingPredicates = [HasStdEnc];
  let ASEPredicate = [HasMSA];
}
```
- EN: Declares reusable TableGen class `MSAPseudo` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSAPseudo`，通常用于抽象共享字段、谓词或编码结构。

### Lines 29-32
```tablegen
class MSA_BIT_B_FMT<bits<3> major, bits<6> minor>: MSAInst {
  bits<5> ws;
  bits<5> wd;
  bits<3> m;
```
- EN: Declares reusable TableGen class `MSA_BIT_B_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_BIT_B_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 34-40
```tablegen
  let Inst{25-23} = major;
  let Inst{22-19} = 0b1110;
  let Inst{18-16} = m;
  let Inst{15-11} = ws;
  let Inst{10-6} = wd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 42-45
```tablegen
class MSA_BIT_H_FMT<bits<3> major, bits<6> minor>: MSAInst {
  bits<5> ws;
  bits<5> wd;
  bits<4> m;
```
- EN: Declares reusable TableGen class `MSA_BIT_H_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_BIT_H_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 47-53
```tablegen
  let Inst{25-23} = major;
  let Inst{22-20} = 0b110;
  let Inst{19-16} = m;
  let Inst{15-11} = ws;
  let Inst{10-6} = wd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 55-58
```tablegen
class MSA_BIT_W_FMT<bits<3> major, bits<6> minor>: MSAInst {
  bits<5> ws;
  bits<5> wd;
  bits<5> m;
```
- EN: Declares reusable TableGen class `MSA_BIT_W_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_BIT_W_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 60-66
```tablegen
  let Inst{25-23} = major;
  let Inst{22-21} = 0b10;
  let Inst{20-16} = m;
  let Inst{15-11} = ws;
  let Inst{10-6} = wd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 68-71
```tablegen
class MSA_BIT_D_FMT<bits<3> major, bits<6> minor>: MSAInst {
  bits<5> ws;
  bits<5> wd;
  bits<6> m;
```
- EN: Declares reusable TableGen class `MSA_BIT_D_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_BIT_D_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 73-79
```tablegen
  let Inst{25-23} = major;
  let Inst{22} = 0b0;
  let Inst{21-16} = m;
  let Inst{15-11} = ws;
  let Inst{10-6} = wd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 81-83
```tablegen
class MSA_2R_FILL_FMT<bits<8> major, bits<2> df, bits<6> minor>: MSAInst {
  bits<5> rs;
  bits<5> wd;
```
- EN: Declares reusable TableGen class `MSA_2R_FILL_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_2R_FILL_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 85-90
```tablegen
  let Inst{25-18} = major;
  let Inst{17-16} = df;
  let Inst{15-11} = rs;
  let Inst{10-6} = wd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 92-94
```tablegen
class MSA_2R_FILL_D_FMT<bits<8> major, bits<2> df, bits<6> minor>: MSAInst {
  bits<5> rs;
  bits<5> wd;
```
- EN: Declares reusable TableGen class `MSA_2R_FILL_D_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_2R_FILL_D_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 96-101
```tablegen
  let Inst{25-18} = major;
  let Inst{17-16} = df;
  let Inst{15-11} = rs;
  let Inst{10-6} = wd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 103-105
```tablegen
class MSA_2R_FMT<bits<8> major, bits<2> df, bits<6> minor>: MSAInst {
  bits<5> ws;
  bits<5> wd;
```
- EN: Declares reusable TableGen class `MSA_2R_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_2R_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 107-112
```tablegen
  let Inst{25-18} = major;
  let Inst{17-16} = df;
  let Inst{15-11} = ws;
  let Inst{10-6} = wd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 114-116
```tablegen
class MSA_2RF_FMT<bits<9> major, bits<1> df, bits<6> minor>: MSAInst {
  bits<5> ws;
  bits<5> wd;
```
- EN: Declares reusable TableGen class `MSA_2RF_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_2RF_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 118-123
```tablegen
  let Inst{25-17} = major;
  let Inst{16} = df;
  let Inst{15-11} = ws;
  let Inst{10-6} = wd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 125-128
```tablegen
class MSA_3R_FMT<bits<3> major, bits<2> df, bits<6> minor>: MSAInst {
  bits<5> wt;
  bits<5> ws;
  bits<5> wd;
```
- EN: Declares reusable TableGen class `MSA_3R_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_3R_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 130-136
```tablegen
  let Inst{25-23} = major;
  let Inst{22-21} = df;
  let Inst{20-16} = wt;
  let Inst{15-11} = ws;
  let Inst{10-6} = wd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 138-141
```tablegen
class MSA_3RF_FMT<bits<4> major, bits<1> df, bits<6> minor>: MSAInst {
  bits<5> wt;
  bits<5> ws;
  bits<5> wd;
```
- EN: Declares reusable TableGen class `MSA_3RF_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_3RF_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 143-149
```tablegen
  let Inst{25-22} = major;
  let Inst{21} = df;
  let Inst{20-16} = wt;
  let Inst{15-11} = ws;
  let Inst{10-6} = wd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 151-154
```tablegen
class MSA_3R_INDEX_FMT<bits<3> major, bits<2> df, bits<6> minor>: MSAInst {
  bits<5> rt;
  bits<5> ws;
  bits<5> wd;
```
- EN: Declares reusable TableGen class `MSA_3R_INDEX_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_3R_INDEX_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 156-162
```tablegen
  let Inst{25-23} = major;
  let Inst{22-21} = df;
  let Inst{20-16} = rt;
  let Inst{15-11} = ws;
  let Inst{10-6} = wd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 164-166
```tablegen
class MSA_ELM_FMT<bits<10> major, bits<6> minor>: MSAInst {
  bits<5> ws;
  bits<5> wd;
```
- EN: Declares reusable TableGen class `MSA_ELM_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_ELM_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 168-172
```tablegen
  let Inst{25-16} = major;
  let Inst{15-11} = ws;
  let Inst{10-6} = wd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 174-176
```tablegen
class MSA_ELM_CFCMSA_FMT<bits<10> major, bits<6> minor>: MSAInst {
  bits<5> rd;
  bits<5> cs;
```
- EN: Declares reusable TableGen class `MSA_ELM_CFCMSA_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_ELM_CFCMSA_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 178-182
```tablegen
  let Inst{25-16} = major;
  let Inst{15-11} = cs;
  let Inst{10-6} = rd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 184-186
```tablegen
class MSA_ELM_CTCMSA_FMT<bits<10> major, bits<6> minor>: MSAInst {
  bits<5> rs;
  bits<5> cd;
```
- EN: Declares reusable TableGen class `MSA_ELM_CTCMSA_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_ELM_CTCMSA_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 188-192
```tablegen
  let Inst{25-16} = major;
  let Inst{15-11} = rs;
  let Inst{10-6} = cd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 194-197
```tablegen
class MSA_ELM_B_FMT<bits<4> major, bits<6> minor>: MSAInst {
  bits<4> n;
  bits<5> ws;
  bits<5> wd;
```
- EN: Declares reusable TableGen class `MSA_ELM_B_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_ELM_B_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 199-205
```tablegen
  let Inst{25-22} = major;
  let Inst{21-20} = 0b00;
  let Inst{19-16} = n{3-0};
  let Inst{15-11} = ws;
  let Inst{10-6} = wd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 207-210
```tablegen
class MSA_ELM_H_FMT<bits<4> major, bits<6> minor>: MSAInst {
  bits<4> n;
  bits<5> ws;
  bits<5> wd;
```
- EN: Declares reusable TableGen class `MSA_ELM_H_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_ELM_H_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 212-218
```tablegen
  let Inst{25-22} = major;
  let Inst{21-19} = 0b100;
  let Inst{18-16} = n{2-0};
  let Inst{15-11} = ws;
  let Inst{10-6} = wd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 220-223
```tablegen
class MSA_ELM_W_FMT<bits<4> major, bits<6> minor>: MSAInst {
  bits<4> n;
  bits<5> ws;
  bits<5> wd;
```
- EN: Declares reusable TableGen class `MSA_ELM_W_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_ELM_W_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 225-231
```tablegen
  let Inst{25-22} = major;
  let Inst{21-18} = 0b1100;
  let Inst{17-16} = n{1-0};
  let Inst{15-11} = ws;
  let Inst{10-6} = wd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 233-236
```tablegen
class MSA_ELM_D_FMT<bits<4> major, bits<6> minor>: MSAInst {
  bits<4> n;
  bits<5> ws;
  bits<5> wd;
```
- EN: Declares reusable TableGen class `MSA_ELM_D_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_ELM_D_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 238-244
```tablegen
  let Inst{25-22} = major;
  let Inst{21-17} = 0b11100;
  let Inst{16} = n{0};
  let Inst{15-11} = ws;
  let Inst{10-6} = wd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 246-249
```tablegen
class MSA_ELM_COPY_B_FMT<bits<4> major, bits<6> minor>: MSAInst {
  bits<4> n;
  bits<5> ws;
  bits<5> rd;
```
- EN: Declares reusable TableGen class `MSA_ELM_COPY_B_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_ELM_COPY_B_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 251-257
```tablegen
  let Inst{25-22} = major;
  let Inst{21-20} = 0b00;
  let Inst{19-16} = n{3-0};
  let Inst{15-11} = ws;
  let Inst{10-6} = rd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 259-262
```tablegen
class MSA_ELM_COPY_H_FMT<bits<4> major, bits<6> minor>: MSAInst {
  bits<4> n;
  bits<5> ws;
  bits<5> rd;
```
- EN: Declares reusable TableGen class `MSA_ELM_COPY_H_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_ELM_COPY_H_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 264-270
```tablegen
  let Inst{25-22} = major;
  let Inst{21-19} = 0b100;
  let Inst{18-16} = n{2-0};
  let Inst{15-11} = ws;
  let Inst{10-6} = rd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 272-275
```tablegen
class MSA_ELM_COPY_W_FMT<bits<4> major, bits<6> minor>: MSAInst {
  bits<4> n;
  bits<5> ws;
  bits<5> rd;
```
- EN: Declares reusable TableGen class `MSA_ELM_COPY_W_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_ELM_COPY_W_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 277-283
```tablegen
  let Inst{25-22} = major;
  let Inst{21-18} = 0b1100;
  let Inst{17-16} = n{1-0};
  let Inst{15-11} = ws;
  let Inst{10-6} = rd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 285-288
```tablegen
class MSA_ELM_COPY_D_FMT<bits<4> major, bits<6> minor>: MSAInst {
  bits<4> n;
  bits<5> ws;
  bits<5> rd;
```
- EN: Declares reusable TableGen class `MSA_ELM_COPY_D_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_ELM_COPY_D_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 290-296
```tablegen
  let Inst{25-22} = major;
  let Inst{21-17} = 0b11100;
  let Inst{16} = n{0};
  let Inst{15-11} = ws;
  let Inst{10-6} = rd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 298-301
```tablegen
class MSA_ELM_INSERT_B_FMT<bits<4> major, bits<6> minor>: MSAInst {
  bits<6> n;
  bits<5> rs;
  bits<5> wd;
```
- EN: Declares reusable TableGen class `MSA_ELM_INSERT_B_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_ELM_INSERT_B_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 303-309
```tablegen
  let Inst{25-22} = major;
  let Inst{21-20} = 0b00;
  let Inst{19-16} = n{3-0};
  let Inst{15-11} = rs;
  let Inst{10-6} = wd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 311-314
```tablegen
class MSA_ELM_INSERT_H_FMT<bits<4> major, bits<6> minor>: MSAInst {
  bits<6> n;
  bits<5> rs;
  bits<5> wd;
```
- EN: Declares reusable TableGen class `MSA_ELM_INSERT_H_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_ELM_INSERT_H_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 316-322
```tablegen
  let Inst{25-22} = major;
  let Inst{21-19} = 0b100;
  let Inst{18-16} = n{2-0};
  let Inst{15-11} = rs;
  let Inst{10-6} = wd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 324-327
```tablegen
class MSA_ELM_INSERT_W_FMT<bits<4> major, bits<6> minor>: MSAInst {
  bits<6> n;
  bits<5> rs;
  bits<5> wd;
```
- EN: Declares reusable TableGen class `MSA_ELM_INSERT_W_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_ELM_INSERT_W_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 329-335
```tablegen
  let Inst{25-22} = major;
  let Inst{21-18} = 0b1100;
  let Inst{17-16} = n{1-0};
  let Inst{15-11} = rs;
  let Inst{10-6} = wd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 337-340
```tablegen
class MSA_ELM_INSERT_D_FMT<bits<4> major, bits<6> minor>: MSAInst {
  bits<6> n;
  bits<5> rs;
  bits<5> wd;
```
- EN: Declares reusable TableGen class `MSA_ELM_INSERT_D_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_ELM_INSERT_D_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 342-348
```tablegen
  let Inst{25-22} = major;
  let Inst{21-17} = 0b11100;
  let Inst{16} = n{0};
  let Inst{15-11} = rs;
  let Inst{10-6} = wd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 350-353
```tablegen
class MSA_I5_FMT<bits<3> major, bits<2> df, bits<6> minor>: MSAInst {
  bits<5> imm;
  bits<5> ws;
  bits<5> wd;
```
- EN: Declares reusable TableGen class `MSA_I5_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_I5_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 355-361
```tablegen
  let Inst{25-23} = major;
  let Inst{22-21} = df;
  let Inst{20-16} = imm;
  let Inst{15-11} = ws;
  let Inst{10-6} = wd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 363-366
```tablegen
class MSA_I8_FMT<bits<2> major, bits<6> minor>: MSAInst {
  bits<8> u8;
  bits<5> ws;
  bits<5> wd;
```
- EN: Declares reusable TableGen class `MSA_I8_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_I8_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 368-373
```tablegen
  let Inst{25-24} = major;
  let Inst{23-16} = u8;
  let Inst{15-11} = ws;
  let Inst{10-6} = wd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 375-377
```tablegen
class MSA_I10_FMT<bits<3> major, bits<2> df, bits<6> minor>: MSAInst {
  bits<10> s10;
  bits<5> wd;
```
- EN: Declares reusable TableGen class `MSA_I10_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_I10_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 379-384
```tablegen
  let Inst{25-23} = major;
  let Inst{22-21} = df;
  let Inst{20-11} = s10;
  let Inst{10-6} = wd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 386-388
```tablegen
class MSA_MI10_FMT<bits<2> df, bits<4> minor>: MSAInst {
  bits<21> addr;
  bits<5> wd;
```
- EN: Declares reusable TableGen class `MSA_MI10_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_MI10_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 390-395
```tablegen
  let Inst{25-16} = addr{9-0};
  let Inst{15-11} = addr{20-16};
  let Inst{10-6} = wd;
  let Inst{5-2} = minor;
  let Inst{1-0} = df;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 397-400
```tablegen
class MSA_VEC_FMT<bits<5> major, bits<6> minor>: MSAInst {
  bits<5> wt;
  bits<5> ws;
  bits<5> wd;
```
- EN: Declares reusable TableGen class `MSA_VEC_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_VEC_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 402-407
```tablegen
  let Inst{25-21} = major;
  let Inst{20-16} = wt;
  let Inst{15-11} = ws;
  let Inst{10-6} = wd;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 409-411
```tablegen
class MSA_CBRANCH_FMT<bits<3> major, bits<2> df>: MSACBranch {
  bits<16> offset;
  bits<5> wt;
```
- EN: Declares reusable TableGen class `MSA_CBRANCH_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_CBRANCH_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 413-417
```tablegen
  let Inst{25-23} = major;
  let Inst{22-21} = df;
  let Inst{20-16} = wt;
  let Inst{15-0} = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 419-421
```tablegen
class MSA_CBRANCH_V_FMT<bits<5> major>: MSACBranch {
  bits<16> offset;
  bits<5> wt;
```
- EN: Declares reusable TableGen class `MSA_CBRANCH_V_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `MSA_CBRANCH_V_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 423-426
```tablegen
  let Inst{25-21} = major;
  let Inst{20-16} = wt;
  let Inst{15-0} = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 428-432
```tablegen
class SPECIAL_LSA_FMT<bits<6> minor>: MSASpecial {
  bits<5> rs;
  bits<5> rt;
  bits<5> rd;
  bits<2> sa;
```
- EN: Declares reusable TableGen class `SPECIAL_LSA_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `SPECIAL_LSA_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 434-440
```tablegen
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-11} = rd;
  let Inst{10-8} = 0b000;
  let Inst{7-6} = sa;
  let Inst{5-0} = minor;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 442-446
```tablegen
class SPECIAL_DLSA_FMT<bits<6> minor>: MSASpecial {
  bits<5> rs;
  bits<5> rt;
  bits<5> rd;
  bits<2> sa;
```
- EN: Declares reusable TableGen class `SPECIAL_DLSA_FMT` for `MipsMSAInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsMSAInstrFormats` 声明可复用的 TableGen 类 `SPECIAL_DLSA_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 448-454
```tablegen
  let Inst{25-21} = rs;
  let Inst{20-16} = rt;
  let Inst{15-11} = rd;
  let Inst{10-8} = 0b000;
  let Inst{7-6} = sa;
  let Inst{5-0} = minor;
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
