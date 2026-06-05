# MicroMipsDSPInstrFormats.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MicroMipsDSPInstrFormats.td`
- Repository: `llvm-project`
- Purpose (EN): Defines `MicroMipsDSPInstrFormats` in LLVM TableGen DSL for the Mips backend, covering instruction encoding formats and bitfield layout.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `MicroMipsDSPInstrFormats`，涵盖指令编码格式与位域布局。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===-- MicroMipsDSPInstrFormats.td - Instruction Formats --*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 9-16
```tablegen
class MMDSPInst<string opstr = "">
    : MipsInst<(outs), (ins), "", [], FrmOther> {
  let ASEPredicate = [HasDSP];
  let EncodingPredicates = [InMicroMips];
  string BaseOpcode = opstr;
  string Arch = "mmdsp";
  let DecoderNamespace = "MicroMips";
}
```
- EN: Declares reusable TableGen class `MMDSPInst` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `MMDSPInst`，通常用于抽象共享字段、谓词或编码结构。

### Lines 18-22
```tablegen
class MMDSPInstAlias<string Asm, dag Result, bit Emit = 0b1>
    : InstAlias<Asm, Result, Emit>, PredicateControl {
  let ASEPredicate = [HasDSP];
  let AdditionalPredicates = [InMicroMips];
}
```
- EN: Declares reusable TableGen class `MMDSPInstAlias` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `MMDSPInstAlias`，通常用于抽象共享字段、谓词或编码结构。

### Lines 24-27
```tablegen
class POOL32A_3R_FMT<string opstr, bits<11> op> : MMDSPInst<opstr> {
  bits<5> rd;
  bits<5> rs;
  bits<5> rt;
```
- EN: Declares reusable TableGen class `POOL32A_3R_FMT` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `POOL32A_3R_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 29-34
```tablegen
  let Inst{31-26} = 0b000000;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-11} = rd;
  let Inst{10-0}  = op;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 36-38
```tablegen
class POOL32A_2R_FMT<string opstr, bits<10> op> : MMDSPInst<opstr> {
  bits<5> rt;
  bits<5> rs;
```
- EN: Declares reusable TableGen class `POOL32A_2R_FMT` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `POOL32A_2R_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 40-45
```tablegen
  let Inst{31-26} = 0b000000;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-6}  = op;
  let Inst{5-0}   = 0b111100;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 47-50
```tablegen
class POOL32A_2RAC_FMT<string opstr, bits<8> op> : MMDSPInst<opstr> {
  bits<5> rt;
  bits<5> rs;
  bits<2> ac;
```
- EN: Declares reusable TableGen class `POOL32A_2RAC_FMT` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `POOL32A_2RAC_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 52-58
```tablegen
  let Inst{31-26} = 0b000000;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-14} = ac;
  let Inst{13-6}  = op;
  let Inst{5-0}   = 0b111100;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 60-63
```tablegen
class POOL32A_3RB0_FMT<string opstr, bits<10> op> : MMDSPInst<opstr> {
  bits<5> rd;
  bits<5> rs;
  bits<5> rt;
```
- EN: Declares reusable TableGen class `POOL32A_3RB0_FMT` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `POOL32A_3RB0_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 65-71
```tablegen
  let Inst{31-26} = 0b000000;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-11} = rd;
  let Inst{10}    = 0b0;
  let Inst{9-0}   = op;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 73-76
```tablegen
class POOL32A_2RSA4_FMT<string opstr, bits<12> op> : MMDSPInst<opstr> {
  bits<5> rt;
  bits<5> rs;
  bits<4> sa;
```
- EN: Declares reusable TableGen class `POOL32A_2RSA4_FMT` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `POOL32A_2RSA4_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 78-83
```tablegen
  let Inst{31-26} = 0b000000;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-12} = sa;
  let Inst{11-0}  = op;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 85-88
```tablegen
class POOL32A_2RSA3_FMT<string opstr, bits<7> op> : MMDSPInst<opstr> {
  bits<5> rt;
  bits<5> rs;
  bits<3> sa;
```
- EN: Declares reusable TableGen class `POOL32A_2RSA3_FMT` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `POOL32A_2RSA3_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 90-96
```tablegen
  let Inst{31-26} = 0b000000;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-13} = sa;
  let Inst{12-6}  = op;
  let Inst{5-0}   = 0b111100;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 98-101
```tablegen
class POOL32A_2RSA5B0_FMT<string opstr, bits<10> op> : MMDSPInst<opstr> {
  bits<5> rt;
  bits<5> rs;
  bits<5> sa;
```
- EN: Declares reusable TableGen class `POOL32A_2RSA5B0_FMT` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `POOL32A_2RSA5B0_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 103-109
```tablegen
  let Inst{31-26} = 0b000000;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-11} = sa;
  let Inst{10}    = 0b0;
  let Inst{9-0}   = op;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 111-114
```tablegen
class POOL32A_2RSA4B0_FMT<string opstr, bits<11> op> : MMDSPInst<opstr> {
  bits<5> rt;
  bits<5> rs;
  bits<4> sa;
```
- EN: Declares reusable TableGen class `POOL32A_2RSA4B0_FMT` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `POOL32A_2RSA4B0_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 116-122
```tablegen
  let Inst{31-26} = 0b000000;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-12} = sa;
  let Inst{11}    = 0b0;
  let Inst{10-0}  = op;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 124-127
```tablegen
class POOL32A_2RSA4OP6_FMT<string opstr, bits<6> op> : MMDSPInst<opstr> {
  bits<5> rt;
  bits<5> rs;
  bits<4> sa;
```
- EN: Declares reusable TableGen class `POOL32A_2RSA4OP6_FMT` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `POOL32A_2RSA4OP6_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 129-135
```tablegen
  let Inst{31-26} = 0b000000;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-12} = sa;
  let Inst{11-6}  = op;
  let Inst{5-0}   = 0b111100;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 137-140
```tablegen
class POOL32A_1RIMM5AC_FMT<string opstr, bits<8> funct> : MMDSPInst<opstr> {
  bits<5> rt;
  bits<5> imm;
  bits<2> ac;
```
- EN: Declares reusable TableGen class `POOL32A_1RIMM5AC_FMT` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `POOL32A_1RIMM5AC_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 142-148
```tablegen
  let Inst{31-26} = 0b000000;
  let Inst{25-21} = rt;
  let Inst{20-16} = imm;
  let Inst{15-14} = ac;
  let Inst{13-6}  = funct;
  let Inst{5-0}   = 0b111100;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 150-153
```tablegen
class POOL32A_2RSA5_FMT<string opstr, bits<11> op> : MMDSPInst<opstr> {
  bits<5> rt;
  bits<5> rs;
  bits<5> sa;
```
- EN: Declares reusable TableGen class `POOL32A_2RSA5_FMT` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `POOL32A_2RSA5_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 155-160
```tablegen
  let Inst{31-26} = 0b000000;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-11} = sa;
  let Inst{10-0}  = op;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 162-165
```tablegen
class POOL32A_1RMEMB0_FMT<string opstr, bits<10> funct> : MMDSPInst<opstr> {
  bits<5> index;
  bits<5> base;
  bits<5> rd;
```
- EN: Declares reusable TableGen class `POOL32A_1RMEMB0_FMT` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `POOL32A_1RMEMB0_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 167-173
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-21} = index;
  let Inst{20-16} = base;
  let Inst{15-11} = rd;
  let Inst{10}    = 0b0;
  let Inst{9-0}   = funct;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 175-177
```tablegen
class POOL32A_1RAC_FMT<string instr_asm, bits<8> funct> : MMDSPInst<instr_asm> {
  bits<5> rs;
  bits<2> ac;
```
- EN: Declares reusable TableGen class `POOL32A_1RAC_FMT` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `POOL32A_1RAC_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 179-185
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-21} = 0;
  let Inst{20-16} = rs;
  let Inst{15-14} = ac;
  let Inst{13-6}  = funct;
  let Inst{5-0}   = 0b111100;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 187-189
```tablegen
class POOL32A_1RMASK7_FMT<string opstr, bits<8> op> : MMDSPInst<opstr> {
  bits<5> rt;
  bits<7> mask;
```
- EN: Declares reusable TableGen class `POOL32A_1RMASK7_FMT` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `POOL32A_1RMASK7_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 191-196
```tablegen
  let Inst{31-26} = 0b000000;
  let Inst{25-21} = rt;
  let Inst{20-14} = mask;
  let Inst{13-6}  = op;
  let Inst{5-0}   = 0b111100;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 198-200
```tablegen
class POOL32A_1RIMM10_FMT<string opstr, bits<10> op> : MMDSPInst<opstr> {
  bits<5> rd;
  bits<10> imm;
```
- EN: Declares reusable TableGen class `POOL32A_1RIMM10_FMT` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `POOL32A_1RIMM10_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 202-207
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-16} = imm;
  let Inst{15-11} = rd;
  let Inst{10}    = 0;
  let Inst{9-0}   = op;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 209-211
```tablegen
class POOL32A_1RIMM8_FMT<string opstr, bits<6> op> : MMDSPInst<opstr> {
  bits<5> rt;
  bits<8> imm;
```
- EN: Declares reusable TableGen class `POOL32A_1RIMM8_FMT` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `POOL32A_1RIMM8_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 213-219
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-21} = rt;
  let Inst{20-13} = imm;
  let Inst{12}    = 0;
  let Inst{11-6}  = op;
  let Inst{5-0}   = 0b111100;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 221-223
```tablegen
class POOL32A_4B0SHIFT6AC4B0_FMT<string opstr, bits<10> op> : MMDSPInst<opstr> {
  bits<6> shift;
  bits<2> ac;
```
- EN: Declares reusable TableGen class `POOL32A_4B0SHIFT6AC4B0_FMT` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `POOL32A_4B0SHIFT6AC4B0_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 225-231
```tablegen
  let Inst{31-26} = 0b000000;
  let Inst{25-22} = 0b0000;
  let Inst{21-16} = shift;
  let Inst{15-14} = ac;
  let Inst{13-10} = 0b0000;
  let Inst{9-0}   = op;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 233-235
```tablegen
class POOL32A_5B01RAC_FMT<string opstr, bits<8> op> : MMDSPInst<opstr> {
  bits<5> rs;
  bits<2> ac;
```
- EN: Declares reusable TableGen class `POOL32A_5B01RAC_FMT` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `POOL32A_5B01RAC_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 237-243
```tablegen
  let Inst{31-26} = 0b000000;
  let Inst{25-21} = 0b00000;
  let Inst{20-16} = rs;
  let Inst{15-14} = ac;
  let Inst{13-6}  = op;
  let Inst{5-0}   = 0b111100;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 245-246
```tablegen
class POOL32I_IMMB0_FMT<string opstr, bits<5> op> : MMDSPInst<opstr> {
  bits<16> offset;
```
- EN: Declares reusable TableGen class `POOL32I_IMMB0_FMT` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `POOL32I_IMMB0_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 248-252
```tablegen
  let Inst{31-26} = 0b010000;
  let Inst{25-21} = op;
  let Inst{20-16} = 0;
  let Inst{15-0}  = offset;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 254-257
```tablegen
class POOL32A_2RBP_FMT<string opstr> : MMDSPInst<opstr> {
  bits<5> rt;
  bits<5> rs;
  bits<2> bp;
```
- EN: Declares reusable TableGen class `POOL32A_2RBP_FMT` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `POOL32A_2RBP_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 259-265
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-14} = bp;
  let Inst{13-6}  = 0b00100010;
  let Inst{5-0}   = 0b111100;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 267-269
```tablegen
class POOL32A_2RB0_FMT<string opstr, bits<10> op> : MMDSPInst<opstr> {
  bits<5> rt;
  bits<5> rs;
```
- EN: Declares reusable TableGen class `POOL32A_2RB0_FMT` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `POOL32A_2RB0_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 271-276
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-10} = 0;
  let Inst{9-0}   = op;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 278-281
```tablegen
class POOL32S_3RB0_FMT<string opstr, bits<10> op> : MMDSPInst<opstr> {
  bits<5> rt;
  bits<5> rs;
  bits<5> rd;
```
- EN: Declares reusable TableGen class `POOL32S_3RB0_FMT` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `POOL32S_3RB0_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 283-289
```tablegen
  let Inst{31-26} = 0b010110;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-11} = rd;
  let Inst{10}    = 0b0;
  let Inst{9-0}   = op;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 291-293
```tablegen
class POOL32A_2R2B0_FMT<string opstr, bits<10> op> : MMDSPInst<opstr> {
  bits<5> rt;
  bits<5> rs;
```
- EN: Declares reusable TableGen class `POOL32A_2R2B0_FMT` for `MicroMipsDSPInstrFormats`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MicroMipsDSPInstrFormats` 声明可复用的 TableGen 类 `POOL32A_2R2B0_FMT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 295-301
```tablegen
  let Inst{31-26} = 0;
  let Inst{25-21} = rt;
  let Inst{20-16} = rs;
  let Inst{15-11} = 0;
  let Inst{10}    = 0;
  let Inst{9-0}   = op;
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
